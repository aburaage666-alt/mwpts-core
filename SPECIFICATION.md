MWPTS Core v0.2
Open Data Layer for Migrant Worker Payslip Transparency

0. Project Metadata and Authority
Item	Value
Framework Name	Migrant Worker Payslip Transparency Standard (MWPTS)
Current Version	Core v0.2 (Pilot Draft - May 2026)
Author / Creator	Kunihisa Koyama (Kuni)
Official Repository	https://github.com/aburaage666-alt/mwpts-core
Official Contact	https://www.linkedin.com/in/kuni-koyama-6566b7105/
Authority Notice	All official updates, JSON schemas, and country profile templates are maintained exclusively at the source repository above. Any fork, implementation, or derivative work should link back to this original source to maintain data integrity.

1. Purpose and publication status
MWPTS Core v0.2 is an open data layer for making migrant workers’ payslips readable, comparable, and suitable for consultation support. It is not an app, not a legal determination engine, and not an official ILO, ISO, or JIS standard.
The Core specification defines jurisdiction-neutral data keys, risk result structures, multilingual explanation tags, and country profile templates. Legal thresholds, wage rates, deduction rules, and support contacts must be maintained in country profiles.
2. Open-source package architecture
Layer	Function	Publication file
Core Schema	Common machine-readable fields for payslip data.	mwpts-core-schema.json
Risk Screening Algorithm	Blank-safe, non-binding review logic with severity and confidence levels.	risk-screening-algorithm.md
Multilingual Intent Framework	Worker-facing explanations for pay, deductions, and review flags.	multilingual-intent-framework.csv
Jurisdiction Profiles	Country-specific wage rules, deduction review thresholds, support contacts, and implementation notes.	jurisdiction-profiles/template.yml and examples
MVP Workbook	Reference spreadsheet implementation for pilots and debugging.	MWPTS_v0.2_MVP_PUBLIC.xlsx

3. Core design principles
•	Core is country-neutral; national rules belong in Country Profiles.
•	Do not require personal identifiers in the Core schema; use a random case_id for screening.
•	Residence or visa status must be optional and treated as sensitive data.
•	Input data, reference data, and risk results must be separated.
•	Risk screening outputs must be labelled as review-required signals, not legal findings.
•	Each risk result should include risk_code, severity, confidence, reason_code, source, and rule_version.
4. Core fields
Key	Type	Requirement	Description
case_id	string	required	Random case ID; avoid names and official ID numbers.
jurisdiction_code	enum	required	JP, KR, TW, SG, VN, or other jurisdiction code.
location_code	string	required when applicable	Region used by the country profile, e.g., JP-13 or VN-R1.
currency_code	enum	required	JPY, KRW, TWD, SGD, VND, etc.
regular_hours	decimal	required	Hours used for wage comparison.
base_wage	decimal	required	Base wage amount before excluded allowances are removed.
excluded_allowances_total	decimal	recommended	Allowances excluded from minimum-wage comparison under country profile rules.
gross_wage_total	decimal	required	Total wage before deductions.
net_wage_paid	decimal	required	Amount received after deductions.
deductions[]	array	required	Deduction entries with category, amount, basis, and explanation status.
has_fixed_overtime_pay	enum	recommended	YES, NO, or UNKNOWN.
fixed_overtime_amount	decimal	conditional	Amount of fixed overtime pay when available.
fixed_overtime_hours_included	decimal	conditional	Number of overtime hours included. Unknown values should trigger GREY review.

5. Core risk screening codes
Risk code	Default severity	Core logic
RISK_MINIMUM_WAGE_REVIEW_REQUIRED	YELLOW/GREY	Compare wage_for_minimum_wage_test with applicable minimum wage only when the country profile enables a wage rule.
RISK_PREMIUM_PAY_REVIEW_REQUIRED	YELLOW/GREY	Blank-safe overtime screening. If fixed overtime exists, route to GREY unless detailed breakdown is available.
RISK_HIGH_OPTIONAL_DEDUCTION_RATIO	YELLOW	Compare optional deductions with base wage or profile-defined denominator.
RISK_DEDUCTION_AUTHORIZATION_REVIEW_REQUIRED	YELLOW	Trigger review when optional deductions exist and authorization/basis is unknown.
RISK_UNEXPLAINED_DEDUCTION_TO_WORKER	YELLOW/GREY	Trigger review when the worker does not understand deductions or understanding is unknown.

6. Five-country implementation model
Profile	Role	Implementation pathway
JP	Domestic pilot and reference implementation	Excel/LINE/support-agency tools. Japan-specific rules are in JP Profile.
KR	Korea EPS-compatible local profile	KakaoTalk bots or support tools can implement a local KR profile.
TW	Traditional Chinese and migrant-worker profile	Web app and labour bureau/broker transparency tools.
SG	Itemised payslip and deduction transparency profile	Focus on payslip explanation and deduction transparency, not general minimum-wage screening.
VN	Language pack and future domestic profile	Vietnamese explanations for destination countries; domestic VN profile can be maintained locally.

10. Licensing, Attribution, and Derivative Implementation Rules
10.1 Licensing and attribution
This specification, including the core data schema, risk-screening algorithms, and multilingual intent framework, is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0). Reusers must provide attribution, link to the original source, indicate changes where applicable, and retain copyright and license notices.
10.2 UI attribution for MWPTS-based implementations
Any software, smartphone application, chatbot, web form, or payroll system that claims to be MWPTS-based or MWPTS-conformant should display the following attribution and active hyperlink within a user-accessible interface such as the About screen, system footer, main menu, or help page.
Required Wording	Link Destination
Powered by MWPTS Core v0.2 Protocol. Designed by Kunihisa Koyama (Kuni). Official Specification & Source: https://github.com/aburaage666-alt/mwpts-core	Official Repository: https://github.com/aburaage666-alt/mwpts-core / Official Contact: https://www.linkedin.com/in/kuni-koyama-6566b7105/

This UI attribution requirement is a project conformance rule for claiming MWPTS-based or MWPTS-conformant implementation status. It is not intended to impose additional restrictions beyond CC BY 4.0 for ordinary reuse of the licensed materials.
10.3 Derivative works and naming policy
Modified core screening algorithms or non-verified jurisdiction rule packs must not be labeled as “Official MWPTS”. Developers should name their systems as derivative works, such as “MWPTS-Based Custom Tool by [Developer Name]”, and clearly include a standard disclaimer that the tool is not a definitive legal determination.
Disclaimer: This material is a pilot transparency framework to help workers and support organizations understand payslip items and identify issues that may require further review. It is not legal advice, legal judgment, administrative finding, or certification.
