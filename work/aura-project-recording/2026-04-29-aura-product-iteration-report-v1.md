<!-- sources:
- Jira LP project: issues assigned to juniper, brandon, parker, biggie, claude, ken; updated 2026-04-21 to 2026-04-29
- Notion PM-TL Sync (Juniper) 2026-04-24: https://www.notion.so/hpcnt/MGAI-PM-TL-Sync-Juniper-34bce00cd354809f91d8fed2c736f9e9 (AURA section)
- Slack #mgai-aura (channel C06FB511ZRR): messages 2026-04-21 to 2026-04-29
-->

[!NOTE] 이 문서 한정으로, 한국어로 작성해.
# AURA Weekly Update — 4/29

[!NOTE] AURA demo eval iteration: eval 4 closing, eval 6 prep 으로 고쳐
## 1. Eval Iteration: Eval 4 Closing → Eval 6 Prep

[!NOTE] rational 설명 자체는 디테일로 분리하고, Eval 5가 skipped 되었다는 사실 전달이 다른 참여자들에게 무슨 말인지 모를테니 그보다는 Knowledge Base를 확장하고 프롬프팅하는 작업의 평가를 이번 주가 아니 다음 주 iteration에서 tone-and-manner 평가와 함께 진행하게 되었다. 왜냐하면 ~ 이렇게 정리해.
Eval 5 (KB concept expansion) is skipped. Biggie flagged on 4/28 that applying the 25-concept KB mid-sprint would require a full prompt restructuring — too many co-dependent changes to ship in one iteration. Juniper agreed. Both the KB update and the prompt restructuring are now scoped to Eval 6, which will evaluate Tinder voice and tone alongside the expanded KB.

[!NOTE] 표 말고 nested된 bullet으로 다시 정리해.
| Task | PoC | Status | Note |
|---|---|---|---|
| Eval 4 labeling & findings | Juniper | Done | Labeling complete. Improvement areas identified: repetition cluster logic, diversity detection, blank photo handling, and UI display issues. |
[!NOTE] 이게 Parker와 Biggie가 같이 한 일이 맞는지 소스로부터 다시 검토해. 아닐텐데?
[!NOTE] 이게 한국 시간 4/29 EOD까지 완료가 됨을 Juniper가 확인하기로 하였다고 적어.
| Eval 4 feedback → demo | Parker, Biggie | In Progress | Prompt v4.6.1 deployed. Changes: repetition target scoped to non-representative photos only; diversity lever simplified to face + full-body = pass; blank/meaningless photos treated as repetition cluster; UI repetition indicator and replacement icon scoped correctly. Juniper reviewing outputs from the road. |
[!NOTE] 이 사이에, https://www.notion.so/hpcnt/MGAI-PM-TL-Sync-351ce00cd35480d6951bffbe26111be7?source=copy_link#351ce00cd354806eacb4c0e550703e38 (Prompt Factory — Eval 인프라 정비) 에 대한 내용이 없으니 그거 채워넣어. Parker가 진행했어.
[!NOTE] 아래 Eval 6 KB 일이 무슨 일인지는 있는데 현황이 없잖아.
| Eval 6 KB expansion | Biggie | In Progress | Goal: integrate 25-concept KB into prompt for Eval 6. KB v2.0 (statistical validation with Cohen's d, Bonferroni, gender×age stratification) is in Prompt Factory. Prompt v5.0.0 restructuring planned in parallel — redundancies, contradictions, and output schema will be cleaned up together before the eval run. |
[!NOTE] 아니 이 업무랑 설명이 안맞잖아. 분류하기 애매한 업무이기는 한데 Eval 6 가 아니라 Tinder on-site 라는 업무 이름으로 넣고, 마지막 product decision 부분은 engineering kickoff쪽과 관련이 더 있으니 아래 섹션으로 내려.
| Eval 6 prep — voice & tone | Juniper | In Progress | On business trip. Running coaching workshop; results will be compiled into team-shared findings (LP-611). Photo Review copy/tone baseline alignment (LP-610) to be completed before Eval 6 starts. Product decisions blocking engineering sync resolved on 4/28 (LP-613). |

[!NOTE] Photo review engineering kickoff 라고 고쳐
## 2. Engineering Kickoff

SOUP Backend capacity opens 5/4. The goal this week is to get the integration contract in front of SOUP engineers before they start.

| Task | PoC | Status | Note |
|---|---|---|---|
[!NOTE] Done 하나는 Serving options decision이고 Brandon이 주도했으며, 이전 notion이나 google docs 확인해서 내용 채워. option 1 v.s. option 2 의사 결정을 위한 문서를 준비하여 Trideep과 논의하고 option 2로 가기로 했다는 결의 내용이어야해. Gemini 3.1 flash lite 모델 자체는 컨펌은 아니고, 만일 성능이 안 나오더라도 MVP는 test region에서 범위 줄여서 실험할 여지가 있다고 적어. 물론 이후에 eng region에서 100% 롤아웃하는 건 그러면 optimization이 완료되어야 해. option 1, 2가 뭔지 간략히 있어야하고.
[!NOTE] 지금 이 entry 자체는 photo review product/engineering decision brief 라고 해야하고, slack에서 brandon의 다양한 의도가 있는데 이 내용에는 그게 잘 드러나지 않아.
| Option 2 Decision Brief | Brandon | Done | "Photo Review Option 2 Kick-off — Decision Brief" written (LP-606). Confirms on-demand Gemini Flash Lite as the serving architecture. The doc has two tracks: Track A covers the ML-side implementation; Track B is the SOUP backend integration spec — the only part SOUP engineers need to review. Full doc is 29 pages; SOUP engineers asked to focus on Track B only. |
[!NOTE] Owen은 제외해도 돼. 내용이 너무 구체적이야. 이 일의 다른 일들과의 관계가 더 잘 드러나게 작성해.
| SOUP engineering review request | Brandon, Owen | In Progress | Owen directed Brandon to open a new Slack thread tagging Ryan Burns and Nathan Lamb, with an explicit callout to review Track B and leave comments on blanks, contract concerns, and implementation decisions. Juniper to follow up at spec kick-off meeting. |
[!NOTE] slack에 보면 나랑 owen이랑 brandon이 같이 이야기해서 위 decision brief 이후에 system design document를 따로 만들겠다는 얘기 있는데 그 항목이 여기에 없어. 이게 Mock server 다음에 바로 있어야해.
| Mock server deployment | Brandon | To Do | Required before SOUP begins implementation. Target: before US Monday 5/4. |
| Timeline | — | — | SOUP starts 5/4. Release QA begins 6/1. Contract and mock server must land before 5/4 to avoid a week of slippage. |

## 3. Demo DLP

[!NOTE] 형식이 일관성이 없어. Parker는 AURA demo, Ken은 research dataset에서 각각 DLP를 적용해야함이 MECE하게 드러나도록 해야지. 그리고 5월 초에 끝나기 때문에 다음 주 중으로 진행할 예정이고, 데이터 삭제가 어려울 거야 없는데 삭제를 염두에 두고 코드를 작성해둔 게 아니다보니 여파를 점검해야한다는 내용을 넣어. 다시 말하지만 숫자는 막 중요한 건 아닌데 eval iteration에 쓰는 160명이 140명으로 줄어들고 그 외에 여파는 없어야 하는 게 이상적이라고 해.
[!NOTE] 내용이 너무 빈약해. 소스를 다시 보고 숫자를 점검한 다음, 숫자 자체는 디테일이므로 nested bullet에 넣던지 하고 이걸 읽는 사람들이 잘 이해하고 검토하고 여파를 인지할 수 있도록 워딩을 고쳐.
Parker identified 164 inactive demo users whose data must be deleted for DLP compliance.

- Ken queried the inactive user list from the database; Parker ran a dry-run confirming the deletion scope: 471 profiles remain after removal, 1,445 data objects and 775 image files to delete.
- Agreed approach: retain users who accessed within the past month (~470 active users); schedule a second DLP pass in 60 days.
- Status: dry-run complete. Awaiting explicit approval before executing.

[!NOTE] 이건 Biggie는 검토만 하고 있어. 얘는 왜 이름이 있는 거야 일관성 없게. 일관성 맞춰.
## 4. rSRR Predictor (Ken, Biggie)

[!NOTE] 이 모델은 MVP 범위는 아니지만 연구 목적으로 학습 중이라고 설명해. MVP 마일스톤 다음 연구 목표를 세우는데 필요하기 때문에 하는 중이라고.
Training for the first rSRR Predictor model is complete. The model uses Qwen3-VL 8B + LoRA with a raw rSRR regression objective — a more direct formulation than the pairwise profile-ranker it replaces.

- Evaluation done: pointwise regression metrics (R², MAE, Pearson), ranking quality (Kendall/Spearman), profile-ranker comparison, and Smart Photo counterfactual alignment check.
[!NOTE] Eval 6의 baseline으로 쓰이는 건지는 잘 모르겠는데? 소스 다시 검토하고, 왜 Trideep하고 공유하려는 지 설명이 없어. 그 이유는 지난 미팅에서 궁금하다고 했기 때문이야.
- Ken proposed the structure for the external report (for Trideep); Biggie reviewing. Document will be shared as a reference baseline for Eval 6.
- Repo cleanup (ldm-models) in progress alongside to prepare for external sharing.
