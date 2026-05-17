# MWPTS v0.2.2 Risk Screening Algorithm

This algorithm is a pilot transparency and triage framework. It does **not** make legal determinations. It identifies payslip items that may require worker explanation, support-agency review, or jurisdiction-specific professional review.

## Official metadata

- Framework Name: Migrant Worker Payslip Transparency Standard (MWPTS)
- Version: Core v0.2.2 Risk Screening Algorithm
- Author / Creator: Kunihisa Koyama (Kuni)
- Official Repository: https://github.com/aburaage666-alt/mwpts-core
- Official Contact: https://www.linkedin.com/in/kuni-koyama-6566b7105/
- License: CC BY 4.0, subject to the attribution and naming rules described in the public package README.

## 0. Canonical field-name policy

Use the following canonical field names in all implementations:

| Canonical field | Deprecated / legacy aliases | Rule |
|---|---|---|
| `hours_regular` | `regular_hours`, `scheduled_hours` | Map legacy aliases to `hours_regular` before validation. |
| `deduction_authorization_status` | `deduction_basis_status` | Map legacy aliases to `deduction_authorization_status` before validation. |

Enum values are uppercase by design. Implementations may normalize lower-case input to uppercase before schema validation, but exported MWPTS-compliant data must use the canonical uppercase values.

## 1. Required jurisdiction profile parameters

A Country Profile should define the following parameters before automated screening is run:

| Parameter | Example | Purpose |
|---|---:|---|
| `minimum_wage_rule_enabled` | `YES` / `NO` | Whether R-01 is active in the jurisdiction. |
| `applicable_minimum_wage` | `1163` | Hourly minimum wage or profile-defined equivalent. |
| `premium_rate` | `1.25` | Overtime premium multiplier or profile-defined premium rule. |
| `tolerance_amount` | `10` | Currency-unit tolerance for rounding and payroll calculation differences. |
| `high_deduction_threshold` | `0.20` | Optional deduction ratio threshold. |
| `deduction_ratio_denominator` | `base_wage` | Default denominator for R-03. |

Country Profiles may define more granular rules for night work, holiday work, fixed overtime, wage inclusion/exclusion, sector-specific rates, or local payroll rounding.

## 2. Severity and confidence

### Severity

| Severity | Meaning |
|---|---|
| `GREEN` | No review signal under the configured profile. |
| `YELLOW` | Review recommended. The item may require explanation, evidence, or professional review. |
| `RED` | Reserved for future high-risk patterns where a profile defines a strong review condition. Avoid legal-determination language. |
| `GREY` | Insufficient data, disabled rule, or ambiguous data. |

### Confidence level table

| source_type | default confidence | Notes |
|---|---:|---|
| `API_PAYROLL` | `HIGH` | Direct payroll-system integration. |
| `CSV_PAYROLL` | `HIGH` | Structured employer/payroll export. |
| `EMPLOYER_SYSTEM` | `HIGH` | Employer-side system data. |
| `PDF` | `MEDIUM` | Official-looking PDF, but extraction may vary. |
| `OCR` | `MEDIUM` | Raise to `HIGH` only after manual verification. Requires `ocr_confidence_score` in the JSON schema. |
| `MANUAL` | `LOW` | Manual entry without evidence. |
| `WORKER_SELF_REPORT` | `LOW` | Useful for triage, not definitive. |
| `OTHER` | `LOW` | Country Profile should define handling. |

## 3. R-01 Minimum wage review

### Purpose

R-01 checks whether the comparable hourly wage used by the Country Profile appears below the applicable minimum wage reference. It is a review signal, not a legal determination.

### Required core fields

- `wage_for_minimum_wage_test`
- `hours_regular`
- `base_wage`
- `excluded_allowances_total` where applicable
- Country Profile: `applicable_minimum_wage`, `minimum_wage_rule_enabled`

### Guard clause

Implementations must never divide by zero. If `hours_regular` is blank, missing, non-numeric, or `<= 0`, return `GREY` before computing the comparable hourly wage.

### Recommended JP-style normalization

```text
IF hours_regular is blank OR hours_regular <= 0 -> GREY
ELSE wage_for_minimum_wage_test = (base_wage - excluded_allowances_total) / hours_regular
```

Country Profiles may define a different normalization formula where local law or payroll practice requires it.

### Decision logic

```text
IF minimum_wage_rule_enabled = NO -> GREY
ELSE IF hours_regular is blank OR hours_regular <= 0 -> GREY
ELSE IF wage_for_minimum_wage_test is blank OR wage_for_minimum_wage_test < 0 -> GREY
ELSE IF wage_for_minimum_wage_test + small_numeric_epsilon < applicable_minimum_wage -> YELLOW
ELSE GREEN
```

### Output code

`RISK_MINIMUM_WAGE_REVIEW_REQUIRED`

### Recommended reason codes

| Reason code | Meaning |
|---|---|
| `MIN_WAGE_RULE_DISABLED` | Country Profile does not enable minimum wage screening. |
| `HOURS_REGULAR_MISSING_OR_ZERO` | Comparable hourly calculation cannot be performed safely. |
| `COMPARABLE_WAGE_MISSING` | `wage_for_minimum_wage_test` is missing or invalid. |
| `BELOW_APPLICABLE_MINIMUM_WAGE` | Comparable hourly wage is below the profile reference. |

## 4. R-02 Premium pay review, including fixed overtime

### Purpose

R-02 checks whether reported premium pay appears inconsistent with overtime hours and the configured Country Profile. Fixed overtime is handled as a transparency review route, not as a legal-validity assessment.

### Required fields

- `hours_overtime`
- `wage_for_minimum_wage_test` or a profile-defined overtime base wage
- `reported_premium_pay` or profile-defined premium pay fields
- `has_fixed_overtime_pay`
- If fixed overtime exists: `fixed_overtime_amount`, `fixed_overtime_hours_included`, `fixed_overtime_breakdown_available`
- Country Profile: `premium_rate`, `tolerance_amount`

### Blank-safe non-fixed-overtime route

```text
IF hours_overtime is blank OR hours_overtime <= 0 -> GREEN
ELSE IF has_fixed_overtime_pay = NO OR has_fixed_overtime_pay = UNKNOWN:
    expected_premium_pay = hours_overtime * wage_for_minimum_wage_test * premium_rate
    reported_premium_pay_safe = SUM(reported_premium_pay)
    IF expected_premium_pay - reported_premium_pay_safe > tolerance_amount -> YELLOW
    ELSE GREEN
```

### Fixed overtime route

The previous simplified rule, “if overtime is within included hours, return GREEN,” is not sufficient. A fixed overtime amount can still be too low when compared with the profile-defined premium-rate baseline. Therefore, fixed overtime requires both **documentation review** and **amount adequacy screening**.

```text
IF has_fixed_overtime_pay = YES:
    IF fixed_overtime_hours_included is blank OR fixed_overtime_hours_included <= 0 -> GREY
    ELSE IF fixed_overtime_amount is blank OR fixed_overtime_amount < 0 -> GREY
    ELSE IF fixed_overtime_breakdown_available != YES -> GREY
    ELSE:
        fixed_overtime_minimum_required = fixed_overtime_hours_included * wage_for_minimum_wage_test * premium_rate

        IF fixed_overtime_amount + tolerance_amount < fixed_overtime_minimum_required -> YELLOW
        ELSE IF hours_overtime is blank OR hours_overtime <= 0 -> GREEN
        ELSE IF hours_overtime <= fixed_overtime_hours_included -> GREEN
        ELSE:
            excess_overtime_hours = hours_overtime - fixed_overtime_hours_included
            expected_excess_premium_pay = excess_overtime_hours * wage_for_minimum_wage_test * premium_rate

            IF reported_excess_premium_pay is available:
                reported_excess_premium_pay_safe = SUM(reported_excess_premium_pay)
            ELSE IF reported_premium_pay is available:
                reported_excess_premium_pay_safe = MAX(SUM(reported_premium_pay) - fixed_overtime_amount, 0)
            ELSE:
                return GREY

            IF expected_excess_premium_pay - reported_excess_premium_pay_safe > tolerance_amount -> YELLOW
            ELSE GREEN
```

Country Profiles may replace `wage_for_minimum_wage_test` with a jurisdiction-specific overtime base wage if local law uses a different calculation base.

### Output code

`RISK_PREMIUM_PAY_INCONSISTENCY`

### Recommended reason codes

| Reason code | Meaning |
|---|---|
| `NO_OVERTIME_HOURS` | No overtime reported. |
| `FIXED_OVERTIME_BREAKDOWN_MISSING` | Fixed overtime exists but included hours or breakdown is missing. |
| `FIXED_OVERTIME_AMOUNT_MISSING` | Fixed overtime amount is missing or invalid. |
| `FIXED_OVERTIME_AMOUNT_BELOW_PROFILE_BASELINE` | Fixed overtime amount is below profile-defined minimum premium baseline. |
| `EXCESS_OVERTIME_PAYMENT_UNCLEAR` | Overtime exceeds included fixed hours, but excess premium pay is unavailable. |
| `EXCESS_OVERTIME_PREMIUM_INCONSISTENT` | Excess overtime premium appears below profile calculation. |

## 5. R-03 Optional deduction ratio

### Purpose

R-03 checks whether optional deductions are high relative to the stable base wage. The default denominator is `base_wage`, not `gross_wage_total`, because gross wages fluctuate with overtime and allowances and can mask high deductions.

### Optional deduction categories

Count the following categories as optional deductions unless a Country Profile defines otherwise:

- `HOUSING`
- `MEALS`
- `TOOLS_UNIFORM`
- `RECRUITMENT_RELATED`
- `DEBT_REPAYMENT`
- `OTHER_OPTIONAL`

### Decision logic

```text
optional_deductions_total = SUM(deductions[].deduction_amount WHERE deduction_category is optional)

IF base_wage is blank OR base_wage <= 0 -> GREY
ELSE optional_deduction_ratio = optional_deductions_total / base_wage
ELSE IF optional_deduction_ratio > high_deduction_threshold -> YELLOW
ELSE GREEN
```

A Country Profile may define a different denominator only if it explicitly documents the reason and the implementation field name.

### Output code

`RISK_HIGH_OPTIONAL_DEDUCTION_RATIO`

### Recommended reason codes

| Reason code | Meaning |
|---|---|
| `BASE_WAGE_MISSING_OR_ZERO` | Ratio cannot be computed safely. |
| `OPTIONAL_DEDUCTION_RATIO_ABOVE_THRESHOLD` | Optional deductions exceed the Country Profile threshold. |

## 6. R-04 Deduction authorization review

### Purpose

R-04 checks whether optional deductions have a declared authorization status.

### Decision logic

```text
IF optional_deductions_total > 0 AND deduction_authorization_status = UNKNOWN -> YELLOW
ELSE GREEN
```

### Output code

`RISK_DEDUCTION_AUTHORIZATION_MISSING`

## 7. R-05 Worker explanation review

### Purpose

R-05 checks whether the worker understands the reason for a deduction. This is a transparency and support-access signal.

### Decision logic

```text
IF deduction exists AND worker_understands = NO -> YELLOW
ELSE IF deduction exists AND worker_understands = UNKNOWN -> GREY
ELSE GREEN
```

### Output code

`RISK_UNEXPLAINED_DEDUCTION_TO_WORKER`

## 8. R-06 Recruitment-related deduction review

### Purpose

R-06 flags recruitment-related deductions or debt-repayment patterns for review under local law, contract documents, and fair-recruitment principles.

### Decision logic

```text
IF deduction_category = RECRUITMENT_RELATED AND deduction_amount > 0 -> YELLOW
ELSE IF deduction_category = DEBT_REPAYMENT AND deduction_amount > 0 -> YELLOW
ELSE GREEN
```

### Output code

`RISK_RECRUITMENT_RELATED_DEDUCTION_REVIEW_REQUIRED`

R-06 does not determine illegality. It signals that recruitment-related costs or debt repayment may require review.

## 9. Output object structure

Each risk result should be exported using the following structure:

```json
{
  "risk_code": "RISK_MINIMUM_WAGE_REVIEW_REQUIRED",
  "severity": "YELLOW",
  "confidence": "MEDIUM",
  "reason_code": "BELOW_APPLICABLE_MINIMUM_WAGE",
  "rule_version": "MWPTS-RISK-v0.2.2"
}
```

## 10. Implementation safeguards

Before running the screening engine, implementations should:

1. Normalize legacy field names to canonical field names.
2. Normalize enum values to uppercase.
3. Treat blank numeric fields as missing, not zero, unless the field is explicitly allowed to be zero.
4. Apply the JSON Schema before running risk rules.
5. Store the Country Profile version used for each result.
6. Export `GREY` instead of raising runtime errors when required data is missing.

## 11. Attribution

Powered by MWPTS Core v0.2 Protocol. Designed by Kunihisa Koyama (Kuni). Official Specification & Source: https://github.com/aburaage666-alt/mwpts-core
