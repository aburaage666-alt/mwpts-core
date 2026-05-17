MWPTS Worker-facing Language Pack v0.2
English / Easy Japanese / Vietnamese / Korean / Traditional Chinese

0. Project Metadata and Authority
Item	Value
Framework Name	Migrant Worker Payslip Transparency Standard (MWPTS)
Current Version	Core v0.2 (Pilot Draft - May 2026)
Author / Creator	Kunihisa Koyama (Kuni)
Official Repository	https://github.com/aburaage666-alt/mwpts-core
Official Contact	https://www.linkedin.com/in/kuni-koyama-6566b7105/
Authority Notice	All official updates, JSON schemas, and country profile templates are maintained exclusively at the source repository above. Any fork, implementation, or derivative work should link back to this original source to maintain data integrity.

1. Purpose
This pack contains worker-facing explanations. It should be reviewed by native speakers and labour-support practitioners before deployment. The goal is intent explanation, not literal word-by-word translation.
2. Core explanation tags
Tag	English	やさしい日本語	Vietnamese	Korean	Traditional Chinese
net_wage	This is the actual amount you receive after deductions.	これは あなたが もらう お金です。	Đây là số tiền thực tế bạn nhận được sau khi trừ các khoản khấu trừ.	이 금액은 공제 후 실제로 받는 금액입니다.	這是扣除各項費用後你實際收到的金額。
deduction_housing	This is a housing or dormitory fee. Check whether it matches your agreement.	これは へやだい です。契約と 同じか かくにんしてください。	Đây là tiền nhà hoặc ký túc xá. Hãy kiểm tra có đúng với thỏa thuận không.	기숙사 또는 주거비입니다. 계약 내용과 같은지 확인하세요.	這是住宿或宿舍費。請確認是否與合約一致。
minimum_wage_review	Your wage may need further review against the applicable minimum wage.	給料が きまった 金額より 少ないかもしれません。相談してください。	Mức lương của bạn có thể cần được kiểm tra theo mức lương tối thiểu.	임금이 최저임금 기준에 맞는지 확인이 필요할 수 있습니다.	你的工資可能需要依最低工資標準進一步確認。
fixed_overtime_review	Fixed overtime may be included. Check how many hours are covered.	固定残業代が あるかもしれません。何時間分か かくにんしてください。	Có thể đã bao gồm tiền làm thêm cố định. Hãy kiểm tra bao gồm bao nhiêu giờ.	고정 연장근로수당이 포함되어 있을 수 있습니다. 몇 시간분인지 확인하세요.	可能包含固定加班費。請確認包含幾小時。
legal_disclaimer	This tool does not make a legal determination.	これは 法律の 判断では ありません。	Công cụ này không phải là kết luận pháp lý.	이 도구는 법적 판단을 하는 것이 아닙니다.	此工具不作法律判斷。
official_source	For the official MWPTS source, see the project repository.	公式のMWPTSは プロジェクトページで かくにんできます。	Vui lòng xem kho lưu trữ chính thức của MWPTS.	공식 MWPTS 자료는 프로젝트 저장소를 확인하세요.	請查看 MWPTS 官方專案來源。

3. Deployment note
For public use, each translation should be reviewed by local practitioners. The English file is the technical reference; worker-facing messages should prioritize clarity, safety, and consultation readiness.
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
