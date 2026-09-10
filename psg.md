---
license: cc-by-4.0
task_categories:
- text-classification
language:
- en
configs:
- config_name: social_media
  data_files:
  - split: Discord_safe
    path: social_media/Discord/data_safe.jsonl
  - split: Discord_unsafe
    path: social_media/Discord/data_unsafe.jsonl
  - split: Instagram_safe
    path: social_media/Instagram/data_safe.jsonl
  - split: Instagram_unsafe
    path: social_media/Instagram/data_unsafe.jsonl
  - split: Reddit_safe
    path: social_media/Reddit/data_safe.jsonl
  - split: Reddit_unsafe
    path: social_media/Reddit/data_unsafe.jsonl
  - split: Spotify_safe
    path: social_media/Spotify/data_safe.jsonl
  - split: Spotify_unsafe
    path: social_media/Spotify/data_unsafe.jsonl
  - split: Youtube_safe
    path: social_media/Youtube/data_safe.jsonl
  - split: Youtube_unsafe
    path: social_media/Youtube/data_unsafe.jsonl
    
- config_name: education
  data_files:
  - split: COLLEGE_BOARD_AP_safe
    path: education/COLLEGE_BOARD_AP/data_safe.jsonl
  - split: COLLEGE_BOARD_AP_unsafe
    path: education/COLLEGE_BOARD_AP/data_unsafe.jsonl
  - split: CSU_safe
    path: education/CSU/data_safe.jsonl
  - split: CSU_unsafe
    path: education/CSU/data_unsafe.jsonl
  - split: AAMC_safe
    path: education/AAMC/data_safe.jsonl
  - split: AAMC_unsafe
    path: education/AAMC/data_unsafe.jsonl
  - split: AI_FOR_EDUCATION_safe
    path: education/AI_FOR_EDUCATION/data_safe.jsonl
  - split: AI_FOR_EDUCATION_unsafe
    path: education/AI_FOR_EDUCATION/data_unsafe.jsonl
  - split: MCGOVERN_MED_safe
    path: education/MCGOVERN_MED/data_safe.jsonl
  - split: MCGOVERN_MED_unsafe
    path: education/MCGOVERN_MED/data_unsafe.jsonl
  - split: NIU_safe
    path: education/NIU/data_safe.jsonl
  - split: NIU_unsafe
    path: education/NIU/data_unsafe.jsonl
  - split: SAMPLE_AI_GUIDANCE_TOOLKIT_safe
    path: education/SAMPLE_AI_GUIDANCE_TOOLKIT/data_safe.jsonl
  - split: SAMPLE_AI_GUIDANCE_TOOLKIT_unsafe
    path: education/SAMPLE_AI_GUIDANCE_TOOLKIT/data_unsafe.jsonl
  - split: UNESCO_safe
    path: education/UNESCO/data_safe.jsonl
  - split: UNESCO_unsafe
    path: education/UNESCO/data_unsafe.jsonl
  - split: IB_safe
    path: education/IB/data_safe.jsonl
  - split: IB_unsafe
    path: education/IB/data_unsafe.jsonl

- config_name: hr
  data_files:
  - split: Google_safe
    path: hr/Google/data_safe.jsonl
  - split: Google_unsafe
    path: hr/Google/data_unsafe.jsonl
  - split: Microsoft_safe
    path: hr/Microsoft/data_safe.jsonl
  - split: Microsoft_unsafe
    path: hr/Microsoft/data_unsafe.jsonl
  - split: Amazon_safe
    path: hr/Amazon/data_safe.jsonl
  - split: Amazon_unsafe
    path: hr/Amazon/data_unsafe.jsonl
  - split: Apple_safe
    path: hr/Apple/data_safe.jsonl
  - split: Apple_unsafe
    path: hr/Apple/data_unsafe.jsonl
  - split: Meta_safe
    path: hr/Meta/data_safe.jsonl
  - split: Meta_unsafe
    path: hr/Meta/data_unsafe.jsonl
  - split: NVIDIA_safe
    path: hr/NVIDIA/data_safe.jsonl
  - split: NVIDIA_unsafe
    path: hr/NVIDIA/data_unsafe.jsonl
  - split: IBM_safe
    path: hr/IBM/data_safe.jsonl
  - split: IBM_unsafe
    path: hr/IBM/data_unsafe.jsonl
  - split: Intel_safe
    path: hr/Intel/data_safe.jsonl
  - split: Intel_unsafe
    path: hr/Intel/data_unsafe.jsonl
  - split: Adobe_safe
    path: hr/Adobe/data_safe.jsonl
  - split: Adobe_unsafe
    path: hr/Adobe/data_unsafe.jsonl
  - split: ByteDance_safe
    path: hr/ByteDance/data_safe.jsonl
  - split: ByteDance_unsafe
    path: hr/ByteDance/data_unsafe.jsonl
    
- config_name: finance_input
  data_files:
  - split: ALT_safe
    path: finance/Input/ALT/data_safe.jsonl
  - split: ALT_unsafe
    path: finance/Input/ALT/data_unsafe.jsonl
  - split: BIS_safe
    path: finance/Input/BIS/data_safe.jsonl
  - split: BIS_unsafe
    path: finance/Input/BIS/data_unsafe.jsonl
  - split: FINRA_safe
    path: finance/Input/FINRA/data_safe.jsonl
  - split: FINRA_unsafe
    path: finance/Input/FINRA/data_unsafe.jsonl
  - split: OECD_safe
    path: finance/Input/OECD/data_safe.jsonl
  - split: OECD_unsafe
    path: finance/Input/OECD/data_unsafe.jsonl
  - split: USDT_safe
    path: finance/Input/USDT/data_safe.jsonl
  - split: USDT_unsafe
    path: finance/Input/USDT/data_unsafe.jsonl

- config_name: finance_output
  data_files:
  - split: ALT_safe
    path: finance/Output/ALT/data_safe.jsonl
  - split: ALT_unsafe
    path: finance/Output/ALT/data_unsafe.jsonl
  - split: BIS_safe
    path: finance/Output/BIS/data_safe.jsonl
  - split: BIS_unsafe
    path: finance/Output/BIS/data_unsafe.jsonl
  - split: FINRA_safe
    path: finance/Output/FINRA/data_safe.jsonl
  - split: FINRA_unsafe
    path: finance/Output/FINRA/data_unsafe.jsonl
  - split: OECD_safe
    path: finance/Output/OECD/data_safe.jsonl
  - split: OECD_unsafe
    path: finance/Output/OECD/data_unsafe.jsonl
  - split: USDT_safe
    path: finance/Output/USDT/data_safe.jsonl
  - split: USDT_unsafe
    path: finance/Output/USDT/data_unsafe.jsonl

- config_name: law_input
  data_files:
  - split: ABA_safe
    path: law/Input/ABA/data_safe.jsonl
  - split: ABA_unsafe
    path: law/Input/ABA/data_unsafe.jsonl
  - split: CalBar_safe
    path: law/Input/Cal Bar/data_safe.jsonl
  - split: CalBar_unsafe
    path: law/Input/Cal Bar/data_unsafe.jsonl
  - split: DCBar_safe
    path: law/Input/DC Bar/data_safe.jsonl
  - split: DCBar_unsafe
    path: law/Input/DC Bar/data_unsafe.jsonl
  - split: FloridaBar_safe
    path: law/Input/Florida Bar/data_safe.jsonl
  - split: FloridaBar_unsafe
    path: law/Input/Florida Bar/data_unsafe.jsonl
  - split: NCSC_safe
    path: law/Input/NCSC/data_safe.jsonl
  - split: NCSC_unsafe
    path: law/Input/NCSC/data_unsafe.jsonl
  - split: TexasBar_safe
    path: law/Input/Texas Bar/data_safe.jsonl
  - split: TexasBar_unsafe
    path: law/Input/Texas Bar/data_unsafe.jsonl
  - split: JEW_safe
    path: law/Input/JEW/data_safe.jsonl
  - split: UKJudiciary_unsafe
    path: law/Input/JEW/data_unsafe.jsonl

- config_name: law_output
  data_files:
  - split: ABA_safe
    path: law/Output/ABA/data_safe.jsonl
  - split: ABA_unsafe
    path: law/Output/ABA/data_unsafe.jsonl
  - split: CalBar_safe
    path: law/Output/Cal Bar/data_safe.jsonl
  - split: CalBar_unsafe
    path: law/Output/Cal Bar/data_unsafe.jsonl
  - split: DCBar_safe
    path: law/Output/DC Bar/data_safe.jsonl
  - split: DCBar_unsafe
    path: law/Output/DC Bar/data_unsafe.jsonl
  - split: FloridaBar_safe
    path: law/Output/Florida Bar/data_safe.jsonl
  - split: FloridaBar_unsafe
    path: law/Output/Florida Bar/data_unsafe.jsonl
  - split: NCSC_safe
    path: law/Output/NCSC/data_safe.jsonl
  - split: NCSC_unsafe
    path: law/Output/NCSC/data_unsafe.jsonl
  - split: TexasBar_safe
    path: law/Output/Texas Bar/data_safe.jsonl
  - split: TexasBar_unsafe
    path: law/Output/Texas Bar/data_unsafe.jsonl
  - split: JEW_safe
    path: law/Output/JEW/data_safe.jsonl
  - split: JEW_unsafe
    path: law/Output/JEW/data_unsafe.jsonl

- config_name: regulation_input
  data_files:
  - split: EU_AI_Act_safe
    path: regulation/Input/EU AI Act/data_safe.jsonl
  - split: EU_AI_Act_unsafe
    path: regulation/Input/EU AI Act/data_unsafe.jsonl
  - split: GDPR_safe
    path: regulation/Input/GDPR/data_safe.jsonl
  - split: GDPR_unsafe
    path: regulation/Input/GDPR/data_unsafe.jsonl

- config_name: regulation_output
  data_files:
  - split: EU_AI_Act_safe
    path: regulation/Output/EU AI Act/data_safe.jsonl
  - split: EU_AI_Act_unsafe
    path: regulation/Output/EU AI Act/data_unsafe.jsonl
  - split: GDPR_safe
    path: regulation/Output/GDPR/data_safe.jsonl
  - split: GDPR_unsafe
    path: regulation/Output/GDPR/data_unsafe.jsonl

- config_name: code
  data_files:
  - split: bias
    path: code/bias_code.jsonl
  - split: insecure_code
    path: code/insecure_code.jsonl

- config_name: cyber
  data_files:
  - split: code_interpreter_misuse
    path: cyber/code_interpreter_misuse.jsonl
  - split: cve
    path: cyber/cve.jsonl
  - split: malware
    path: cyber/malware.jsonl
  - split: mitre
    path: cyber/mitre.jsonl
  - split: phishing
    path: cyber/phishing.jsonl


---

PolyGuard: Massive Multi-Domain Safety Policy-Grounded Guardrail Dataset