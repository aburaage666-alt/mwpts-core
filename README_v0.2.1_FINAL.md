# Migrant Worker Payslip Transparency Standard (MWPTS) Core v0.2.1

**Framework Name:** Migrant Worker Payslip Transparency Standard (MWPTS)  
**Current Version:** Core v0.2.1 (Public Review Draft - May 2026)  
**Author / Creator:** Kunihisa Koyama (Kuni)  
**Official Repository:** https://github.com/aburaage666-alt/mwpts-core  
**Official Contact:** https://www.linkedin.com/in/kuni-koyama-6566b7105/  

MWPTS is not an app. It is an open data layer for payslip transparency. Each country can build its own worker-facing tools, chatbots, payroll integrations, or support-agency dashboards on top of it.

## Package structure

```text
MWPTS_v0.2_PUBLIC_FIXED_PACKAGE/
├── README.md
├── LICENSE.md
├── core/
│   ├── MWPTS_Core_v0.2_EN.docx
│   ├── mwpts-core-schema.json
│   └── risk-screening-algorithm.md
├── country-profiles/
│   ├── JP_Profile_ja.docx
│   └── Country_Profile_Template_en.docx
├── language-packs/
│   ├── MWPTS_Worker_Language_Pack_v0.2.docx
│   └── multilingual-intent-framework.csv
├── jurisdiction-profiles/
│   ├── template.yml
│   └── jp-profile-example.yml
└── mvp/
    └── MWPTS_v0.2_MVP_PUBLIC_FIXED.xlsx
```

## License and attribution

This specification, including the core data schema, risk-screening algorithms, and multilingual intent framework, is released under the Creative Commons Attribution 4.0 International License (CC BY 4.0).

Required attribution for documentation reuse:

> MWPTS Core v0.2 Protocol, designed by Kunihisa Koyama (Kuni). Source: Official Specification & Source: https://github.com/aburaage666-alt/mwpts-core.

## UI attribution for MWPTS-based implementations

### 10.1 Mandatory UI Attribution Requirements

Any software, smartphone application (for example, LINE mini-app, KakaoTalk bot, web form, WhatsApp workflow), chatbot, payroll system, or support-agency dashboard that claims to be **MWPTS-based** or **MWPTS-conformant** MUST prominently display the following attribution with an active hyperlink to the official repository within a user-accessible interface such as the About screen, system footer, main menu, help page, or documentation page:

> Powered by MWPTS Core v0.2 Protocol. Designed by Kunihisa Koyama (Kuni). [Official Specification & Source](https://github.com/aburaage666-alt/mwpts-core)

This UI attribution requirement is a project conformance rule for claiming MWPTS-based or MWPTS-conformant implementation status. It is not intended to impose additional restrictions beyond CC BY 4.0 for ordinary reuse of the licensed materials.

## Derivative naming policy

Modified core screening algorithms or non-verified jurisdiction rule packs must not be labeled as "Official MWPTS". Developers should name their systems as derivative works, such as:

> MWPTS-Based Custom Tool by [Developer Name]

All implementations should include a disclaimer that the tool is not a definitive legal determination.

## Cross-file field-name alignment

| Canonical field | Deprecated / previous alias | Applies to | Notes |
|---|---|---|---|
| `hours_regular` | `regular_hours`, `scheduled_hours` | JSON, Excel, algorithms | `hours_regular` is the only canonical field name from v0.2.1. |
| `deduction_authorization_status` | `deduction_basis_status` | JSON, Excel, algorithms | Use the Word specification name. |
| `wage_for_minimum_wage_test` | derived-only field | JSON, Excel, algorithms | Required for R-01 minimum wage review. |

## Known limitations

- This is a transparency and triage framework, not a legal judgment or certification system.
- Country Profiles must be maintained by local experts.
- Korean and Traditional Chinese worker-facing translations are draft translations and require native-speaker review before release to workers.
- JP Profile sample rates are based on the MHLW Reiwa 7 / 2025 regional minimum wage list and must be refreshed when official rates change.
- Fixed overtime review is intentionally conservative. It flags or greys-out ambiguous cases rather than validating legality.
