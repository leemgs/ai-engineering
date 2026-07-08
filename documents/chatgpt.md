# AI 엔지니어링의 4가지 레이어 — ChatGPT (챗지피티) 편

AI 시스템을 구축하고 고도화할 때 고려해야 하는 **4가지 핵심 레이어(프롬프트, 컨텍스트, 하네스, 루프)**의 개념과 오픈에이아이(OpenAI)의 **챗지피티(ChatGPT / GPT)**를 활용한 실제 구현 예시를 정리한 문서입니다.

각 레이어는 이전 레이어를 감싸며 AI 시스템의 안정성, 정확성, 자동화 수준을 높여줍니다.

---

## 📌 4가지 레이어 한눈에 보기

| 레이어 | 챗지피티(ChatGPT) 관점의 의미 | 비유 | 주요 역할 |
| :--- | :--- | :--- | :--- |
| **1. 프롬프트 (Prompt)** | 사용자가 입력하는 직접적인 명령어 | **씨앗** (핵심 알맹이) | AI에게 무엇을 묻고 요청할지 결정 |
| **2. 컨텍스트 (Context)** | System Message(Role), 배경 지식, 제약 조건 | **토양** (자라나는 배경) | AI가 참고할 맥락과 출력 형식 지정 |
| **3. 하네스 (Harness)** | 파이썬 코드로 감싼 입력/출력 검증 및 예외 처리 | **화분** (모양을 잡아줌) | 에러 방지, 구조화된 데이터(JSON) 강제 |
| **4. 루프 (Loop)** | 에이전트 구조의 자가 검증 및 반복 수정 | **자동 급수 장치** (스스로 순환) | 목표 달성까지 [생성 ➔ 평가 ➔ 수정] 반복 |

---

## 1. 프롬프트 (Prompt) ── *알맹이 채우기*
가장 기본이 되는 레이어로, AI 모델에게 직접적으로 내리는 명령어 또는 질문입니다.

* **개념:** 채팅창에 정제되지 않은 질문을 툭 던지는 단계입니다.
* **실전 예시 (유저 입력):**
    > "나 오늘 기분이 좀 우울한데 읽을 만한 책 추천해 줘."
* **한계:** AI는 사용자의 성향, 원하는 어조, 배경 상황을 알지 못하므로 뻔하거나 평범한 답변(예: 일반적인 자기계발서 추천)을 출력할 확률이 높습니다.

---

## 2. 컨텍스트 (Context) ── *맥락과 배경 주입*
프롬프트 주변에 AI가 참고할 수 있는 **배경 정보, 제약 조건, 예시(Few-shot)** 등을 감싸주는 단계입니다. 챗지피티의 `System Message(role: "system")` 기능을 활용해 설정합니다.

* **개념:** AI에게 페르소나를 부여하고 가이드라인을 제공하여 답변의 일관성을 확보합니다.
* **실전 예시 (System Message 설정):**
    ```text
    [역할] 너는 따뜻하고 공감 능력이 뛰어난 전문 사서야.
    [참고 데이터/지식] 
    - 우울할 때: 에세이 '죽고 싶지만 떡볶이는 먹고 싶어', 소설 '불편한 편의점'
    - 불안할 때: 심리학 책 '어른의 마음 공부'
    [출력 형식] 반드시 책 제목, 저자, 그리고 추천 이유를 3줄 이내로 다정하게 작성해줘.
    ```
* **결과:** 유저가 똑같이 "우울해"라고 입력해도, AI는 설정된 맥락(Context)을 바탕으로 훨씬 정돈되고 맞춤화된 답변을 제공합니다.

---

## 3. 하네스 (Harness) ── *안전장치와 도구 연결*
AI 모델이 서비스 내에서 안정적으로 작동하도록 돕는 **코드적인 안전장치(파이프라인)**입니다. 챗지피티 API를 파이썬(Python) 등의 코드로 감싸서 제어합니다.

* **개념:** 입력값 검증, 예외 처리, 결과물의 구조화(JSON 변환 강제) 및 실패 시 재시도(Retry) 로직을 포함합니다.
* **실전 예시 (Python 가상 코드):**
    ```python
    from openai import OpenAI
    import json

    client = OpenAI()

    user_input = "나 오늘 우울해"

    # [하네스 기능 1] 입력값 검증
    if len(user_input) < 2:
        print("질문이 너무 짧습니다.")
        exit()

    # 챗지피티 API 호출 (컨텍스트와 프롬프트 결합)
    # response_format으로 JSON 출력을 강제할 수 있습니다.
    response = client.chat.completions.create(
        model="gpt-4o",
        response_format={"type": "json_object"},
        messages=[
            {"role": "system", "content": "너는 책 추천 봇이야. 답변은 반드시 JSON 형식 {'book': '제목', 'reason': '이유'}로만 해."},
            {"role": "user", "content": user_input},
        ],
    )

    # [하네스 기능 2] 출력 검증 및 예외 처리 (JSON 파싱 에러 방지)
    try:
        result = json.loads(response.choices[0].message.content)
        print(f"추천 도서: {result['book']}")
    except json.JSONDecodeError:
        # 챗지피티가 형식을 어겼을 때 에러를 터뜨리지 않고 부드럽게 대처하는 로직
        print("AI가 형식을 지키지 않아 재요청 등의 예외 처리를 수행합니다.")
    ```

---

## 4. 루프 (Loop) ── *스스로 판단하고 반복*
AI가 단발성 답변으로 끝내는 것이 아니라, 특정 기준이나 목표를 달성할 때까지 **[생성 ➔ 평가 ➔ 수정]의 과정을 스스로 반복(에이전트 구조)**하게 만드는 최종 레이어입니다.

* **개념:** "내가 만든 결과물이 완벽한가?"를 AI가 스스로 검토(Self-Correction)하고 기준을 통과할 때까지 순환합니다.
* **실전 예시 (Python 구현 코드):**
    ```python
    from openai import OpenAI
    import json

    client = OpenAI()

    def ask_gpt(system_prompt, user_message):
        response = client.chat.completions.create(
            model="gpt-4o",
            response_format={"type": "json_object"},
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": user_message},
            ],
        )
        return response.choices[0].message.content

    user_complaint = "나 오늘 너무 무기력하고 우울해. 위로가 되는 책 추천해줘. (주의: 뻔한 자기계발서는 절대 싫어)"
    system_context = "너는 따뜻한 사서야. 출력을 반드시 JSON 형식({'book_title': '제목', 'reason': '이유'})으로 해줘."

    # 루프 제어 변수 (무한 루프 방지)
    max_turns = 3
    current_turn = 0

    # 1차 초안 생성
    draft_recommendation = ask_gpt(system_context, user_complaint)

    while current_turn < max_turns:
        current_turn += 1
        print(f"🔄 [루프 {current_turn}회차] 챗지피티의 답변을 스스로 검증 중...")

        # 자가 검증을 위한 비판자(Critic) 프롬프트 구성
        critic_prompt = f"""
        너는 엄격한 품질 검수관이야. 방금 AI 사서가 작성한 답변이 사용자의 요구사항을 만족하는지 평가해줘.
        [사용자 요구사항]: {user_complaint}
        [AI 사서의 답변]: {draft_recommendation}

        특히 '자기계발서'를 추천했는지 철저히 확인해라. 결과는 반드시 아래 JSON 형식으로만 응답해:
        {{
          "is_passed": true 또는 false,
          "feedback": "거절 사유 또는 통과 사유"
        }}
        """

        critic_result_raw = ask_gpt("너는 엄격한 검수관이야. JSON으로만 답해.", critic_prompt)
        critic_result = json.loads(critic_result_raw)

        if critic_result["is_passed"]:
            print("✅ 검수 통과! 사용자에게 최종 답변을 전송합니다.\n")
            break
        else:
            print(f"❌ 검수 탈락! 사유: {critic_result['feedback']}")
            print("💡 답변을 수정하기 위해 루프를 재가동합니다.\n")

            # 피드백을 반영하여 초안 업데이트 후 루프 재진입
            fix_prompt = f"이전 답변 거절 사유: {critic_result['feedback']}. 이를 반영해서 다시 추천해줘."
            draft_recommendation = ask_gpt(system_context, fix_prompt)

    # 최종 결과 출력
    final_output = json.loads(draft_recommendation)
    print(f"📚 최종 추천 도서: {final_output['book_title']}")
    ```

---

## 💡 요약 및 결론
- **프롬프트와 컨텍스트**만 사용하면 AI가 가끔 주의사항을 누락하거나 환각(Hallucination) 현상을 보여 실패한 답변이 유저에게 그대로 노출될 수 있습니다.
- **하네스와 루프** 레이어를 결합하여 파이프라인을 구축하면, 유저에게 전달되기 전에 시스템 내부에서 AI가 알아서 스크리닝하고 완벽하게 정제된 결과물만 보장할 수 있습니다.

---

## 🔎 ChatGPT의 특징
- **범용성과 생태계:** 방대한 사용자층과 플러그인/GPTs, Assistants API 등 풍부한 생태계를 보유합니다.
- **구조화 출력(Structured Outputs):** `response_format`과 JSON Schema를 통해 출력 형식을 엄격하게 보장할 수 있습니다.
- **강력한 Function Calling:** 외부 도구/함수 호출 기능이 성숙하여 하네스 및 에이전트 구성에 유리합니다.
