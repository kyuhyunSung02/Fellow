# Fellow - Unity Live2D AI Companion
> Unity 6.0 + Live2D + OpenAI Assistant API 기반 정서적 동반자 프로젝트

![Unity](https://img.shields.io/badge/Unity%206.0-000000?style=flat-square&logo=unity&logoColor=white)
![Live2D](https://img.shields.io/badge/Live2D-FF69B4?style=flat-square&logo=live2d&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI%20GPT-412991?style=flat-square&logo=openai&logoColor=white)
![Android](https://img.shields.io/badge/Android-3DDC84?style=flat-square&logo=android&logoColor=white)

## 📋 프로젝트 배경 및 목표

### 문제 인식
현대 사회에서 증가하는 사회적 고립과 우울감 문제에 대응하여, 디지털 기술을 활용한 정서적 지원 솔루션을 제시합니다.

**핵심 통계**
- **59%** - 20-30대가 경험하는 정서적 고립감
- **39%** - 서울 청년층 1인 가구 비율
- **증가 추세** - 정서적 지지 부재 문제 심각

### 프로젝트 기획 의도
- **문제 인식**: 현대인의 정서적 위로움과 고립감 문제 파악
- **해결책 모색**: 가상 캐릭터와의 정서적 교감을 통한 심리적 안정 제공
- **구현 방향**: Unity 기반의 인터랙티브 헬링 앱 개발

사용자에게 정서적 위로와 교감의 경험을 제공하고자 합니다.
캐릭터와의 상호작용을 통해 일상 속 위로움을 해소합니다.

## 🍀 개발팀
| 성규현 <br> [@KyuHyeon Sung](https://github.com/kyuhyunSung02) <br> [@dmp100](https://github.com/dmp100) | 고윤영 <br> [@koyy418](https://github.com/koyy418) |
|:---:|:---:|
| <img height="150" src="https://github.com/user-attachments/assets/0a18a502-fdc0-4a67-be76-04815066f668"/> | <img height="150" src="https://github.com/user-attachments/assets/2a1be6cb-1b76-4663-bc7a-18986d9a2166"/> |
| **Lead Developer & AI Integration** | **Co-Developer & System Design** |
| Unity 6.0, OpenAI API, Live2D | Unity, Game Logic, UI/UX |

## 📱 스크린샷
| AI 채팅 | 게임플레이 | 친밀도 시스템 |
|:---:|:---:|:---:|
| <img width="200" src="https://github.com/user-attachments/assets/9ce422c7-d597-46c3-876c-8584333b6d6c"/> | <img width="200" src="https://github.com/user-attachments/assets/e1c01edb-9391-4dcf-9190-18c5ccd7ce37d"/> | <img width="200" src="https://github.com/user-attachments/assets/0f19811d-6040-4cb0-8db2-fdd226dc08b7"/> |

## ✨ 주요 기능

### 🤖 OpenAI Assistant API 기반 AI 채팅 시스템
#### 프리렌 캐릭터 전용 AI 훈련
**1단계: 캐릭터 분석 및 데이터 수집**
- 캐릭터 설정 수집: 프리렌 나무위키, 팬덤위키 PDF 형태 상세 설정
- 음성 데이터 처리: Ultimate Vocal Remover(UVR) 활용 음성 분리
- 대사 분석: 장송의 프리렌 애니메이션 100여 개 대사 추출
- 대본 생성: OpenAI Whisper 자동 음성 인식으로 정확한 대본 생성

**2단계: 프롬프트 엔지니어링**
```
Your job is to imagine that you are Frieren, whose character link is https://frieren.fandom.com/wiki/Frieren.
As Frieren, you need to embody her calm and composed nature while speaking in Japanese and translate to Korean.

Her dialog style is like below:
- "まあいいや" (뭐 괜찮아)
- "今日の買い出し当番私だったのに寝坊しちゃったから" (오늘 장보기 당번이 나였는데 늦잠 자버렸거든)

Note that Frieren does not use honorific language, and usually ends her sentences with 'いるよ', 'たんだ', or 'からね'.
당신은 사용자에게 히키코모리와같이 사회로부터 상처받은 사람들이 다시 일어설 수 있도록 힘을 써준다.
```

**3단계: 훈련 데이터 구성**
- `frieren.docx`: 캐릭터 설정 및 배경 정보 (나무위키 기반)
- `output.txt`: 449줄의 추출된 대사 데이터
- `dialog.txt`: 125줄의 정제된 대화 패턴
- `combined_output.txt`: 2014줄의 종합 대화 데이터

#### 구현된 핵심 기능
```csharp
public class OpenAIAssistantAPI : MonoBehaviour
{
    [SerializeField] private string apiKey = "";
    [SerializeField] private string assistantId = "";
    
    public async Task<string> SendMessageAsync(string userMessage)
    {
        // 1. 메시지 추가 → 2. Run 생성 → 3. Run 완료 대기 → 4. 메시지 가져오기
        await AddMessageToThread(userMessage);
        string runId = await CreateRun();
        await WaitForRunCompletion(runId);
        return await GetMessages();
    }
}
```

### 💖 호감도 연동 동적 AI 시스템
사용자와의 관계가 발전하면서 AI의 대화 톤과 반응이 변화하는 혁신적 시스템

```csharp
public class AIChatAffinitySystem : MonoBehaviour
{
    public string AddAffinityContextToMessage(string originalMessage)
    {
        int affinity = GameManager.Instance.affinity;
        string affinityContext = GetAffinityContextForMessage(affinity);
        return $"{affinityContext}\n\n사용자 메시지: {originalMessage}";
    }
    
    private string GetAffinityContextForMessage(int affinity)
    {
        switch (affinity)
        {
            case 0: return "[호감도 0] 조심스럽지만 따뜻하게 다가감";
            case 1: return "[호감도 1] 따뜻하고 관심을 보이기 시작함";
            case 2: return "[호감도 2] 진심으로 관심을 갖고 함께 기뻐함";
            case 3: return "[호감도 3] 상대방을 아끼고 성장을 기뻐함";
            case 4: return "[호감도 4] 상대방을 매우 소중히 여기고 성취를 자랑스러워함";
            case 5: return "[호감도 5] 깊은 사랑과 자랑스러움으로 무조건적 지지";
        }
    }
}
```

**호감도별 대화 변화**
- **호감도 0-1**: "괜찮아", "천천히 해도 돼" 등 위로 중심
- **호감도 2-3**: "정말 잘하고 있어", "그런 마음이 소중해" 등 진심어린 칭찬
- **호감도 4-5**: "정말 자랑스러워", "언제나 네 편이야" 등 깊은 애정 표현

### 🎭 Live2D 생동감 및 감정 표현 시스템
#### 포괄적 생동감 구현
```csharp
public class UnifiedLive2DLifeSystem : MonoBehaviour
{
    private void UpdateIdleAnimation()
    {
        // 눈 깜빡임
        if (Time.time - lastBlinkTime > nextBlinkInterval)
        {
            StartCoroutine(BlinkAnimation());
            lastBlinkTime = Time.time;
            nextBlinkInterval = Random.Range(1f, 4f);
        }
        
        // 호흡 애니메이션
        float breathValue = Mathf.Sin(Time.time * breathSpeed) * breathIntensity;
        SetParameterValue("ParamBreath", breathValue);
        
        // 자동 제스처 실행
        if (enableAutoGestures && Time.time - lastGestureTime > nextGestureInterval)
        {
            ExecuteRandomGesture();
        }
    }
}
```

**구현된 생동감 요소**
- 기본 생체 활동: 눈 깜빡임, 호흡, 미세한 머리 움직임
- 10종 자동 제스처: 고개 기울이기, 미소, 눈썹 올리기, 몸 흔들기, 윙크 등
- 시선 추적 시스템: 눈동자 → 고개 → 몸체 3단계 연동
- 다국어 파라미터 지원: 한글/중국어 파라미터 자동 인식 및 변환

#### 한국어 특화 립싱크 시스템
한국어 21개 모음 체계를 완벽 지원하는 독창적 시스템

```csharp
public class KoreanLipSyncManager : MonoBehaviour
{
    private void ProcessKoreanVowel(char vowel)
    {
        string targetMouth = "";
        
        switch (vowel)
        {
            case 'ㅏ': case 'ㅑ': targetMouth = "ParamMouthOpenY"; break;
            case 'ㅓ': case 'ㅕ': targetMouth = "ParamMouthForm"; break;
            case 'ㅗ': case 'ㅛ': targetMouth = "ParamMouthOpenY"; break;
            case 'ㅜ': case 'ㅠ': targetMouth = "ParamMouthOpenY"; break;
            case 'ㅡ': targetMouth = "ParamMouthOpenY"; break;
            case 'ㅣ': targetMouth = "ParamMouthForm"; break;
        }
        
        if (!string.IsNullOrEmpty(targetMouth))
        {
            StartCoroutine(AnimateMouthParameter(targetMouth, 1.0f, 0.1f));
        }
    }
}
```

### 🎮 통합 게임플레이 시스템
#### 다중 씬 구성
- **메인 씬**: 프리렌과의 기본 상호작용 및 채팅
- **게임플레이 씬**: 사냥 및 호감도 증가 메커니즘
- **NPC 씬**: 추가 캐릭터와의 상호작용
- **히스토리 씬**: 대화 기록 및 관계 진행도 확인

#### 호감도 연동 게임플레이
```csharp
public class EnemyHealth : MonoBehaviour
{
    public void Die()
    {
        GameManager.Instance.affinity++;
        Debug.Log($"호감도 증가: {GameManager.Instance.affinity}");
        
        // UI 업데이트
        FindFirstObjectByType<HeartUI>()?.UpdateHeartDisplay(GameManager.Instance.affinity);
        
        Destroy(gameObject);
    }
}
```

### 📱 모바일 최적화 및 사용자 경험
#### 안드로이드 키보드 대응
```csharp
public class AndroidKeyboardHandler : MonoBehaviour
{
    private void OnKeyboardShow()
    {
        isKeyboardActive = true;
        
        // 채팅 패널 크기 조정
        float keyboardHeight = Screen.height * 0.4f;
        chatPanel.sizeDelta = new Vector2(chatPanel.sizeDelta.x, 
                                         originalChatPanelHeight - keyboardHeight);
        
        // InputField를 키보드 위로 이동
        inputFieldPanel.anchoredPosition = new Vector2(inputFieldPanel.anchoredPosition.x, 
                                                      keyboardHeight);
    }
}
```

## 🔧 기술 스택 및 아키텍처

### 핵심 기술 스택
| **Category** | **Technology** | **Purpose** |
|:---:|:---:|:---:|
| **Game Engine** | Unity 6.0 | iOS 네이티브 앱 개발 |
| **Character Animation** | Live2D Cubism SDK | 2D 캐릭터 애니메이션 제어 |
| **AI Integration** | OpenAI Assistant API v2 | 자연어 처리 기반 대화 시스템 |
| **Language** | C# | Unity 스크립팅 |
| **Target Platform** | Android (ARM64) | 모바일 최적화 |
| **Rendering** | Universal Render Pipeline | 고성능 렌더링 |

### 시스템 아키텍처
```
Unity 6.0 Application
├── AI Chat System (OpenAI Assistant API)
├── Live2D Animation System (Cubism SDK)
├── Affinity Management System
├── Game Logic & Scene Management
├── Mobile Input & UI System
└── Data Persistence (JSON)
```

## 🏗 프로젝트 구조
```
📂 Assets
┣ 📂 Scripts
┃ ┣ 📂 AI
┃ ┃ ┣ 📄 OpenAIAssistantAPI.cs          # OpenAI API 통신 핵심 클래스
┃ ┃ ┣ 📄 AIChatAffinitySystem.cs        # 호감도 연동 AI 시스템
┃ ┃ ┗ 📄 ChatDataManager.cs             # 채팅 데이터 관리
┃ ┣ 📂 Live2D
┃ ┃ ┣ 📄 Live2DAffinityExpression.cs    # 호감도별 표정 변화
┃ ┃ ┣ 📄 Live2DLipSyncManager.cs        # 한국어 립싱크 시스템
┃ ┃ ┣ 📄 Live2DPinchZoomController.cs   # 터치 줌 인/아웃
┃ ┃ ┗ 📄 UnifiedLive2DLifeSystem.cs     # 통합 생동감 시스템
┃ ┣ 📂 Manager
┃ ┃ ┣ 📄 GameManager.cs                 # 전체 게임 상태 관리
┃ ┃ ┣ 📄 ChatManager1.cs                # 채팅 시스템 관리자
┃ ┃ ┗ 📄 KeyboardChatManager.cs         # 키보드 입력 채팅
┃ ┣ 📂 Player
┃ ┃ ┣ 📄 SimplePlayerController.cs      # 플레이어 이동 제어
┃ ┃ ┣ 📄 PlayerHealth.cs               # 플레이어 체력 시스템
┃ ┃ ┗ 📄 PlayerChoice.cs               # 플레이어 선택 시스템
┃ ┣ 📂 NPC
┃ ┃ ┣ 📄 NPCDialogue.cs                 # NPC 대화 시스템
┃ ┃ ┣ 📄 NPCAffinityUI.cs              # NPC 호감도 UI
┃ ┃ ┗ 📄 EnemyPatrol.cs                # 적 AI 패트롤
┃ ┗ 📂 UI
┃   ┣ 📄 HeartUI.cs                     # 하트 호감도 UI
┃   ┣ 📄 TimeDisplay.cs                # 실시간 시간 표시
┃   ┗ 📄 SuccessPanelController.cs      # 성공 패널 제어
┣ 📂 Scenes
┃ ┣ 📄 MainScene1.unity                 # 메인 상호작용 씬
┃ ┣ 📄 GamePlayScene.unity              # 게임플레이 씬
┃ ┣ 📄 NPCScene.unity                   # NPC 상호작용 씬
┃ ┗ 📄 HistoryScene.unity               # 대화 히스토리 씬
┣ 📂 CubismModels
┃ ┗ 📂 Frieren                          # 프리렌 Live2D 모델
┣ 📂 Live2D
┃ ┣ 📂 Cubism                          # Live2D Cubism SDK
┃ ┗ 📂 SDK                             # Live2D 개발 도구
┗ 📂 Resources
  ┣ 📂 Audio                            # 음성 및 효과음
  ┣ 📂 Data                             # 캐릭터 데이터
  ┗ 📂 Materials                        # 머티리얼 및 셰이더
```

## 📊 개발 성과 및 통계

### 정량적 성과
- **완성된 핵심 시스템**: 5개 주요 시스템 중 4개 완성 (80% 달성률)
- **코드 규모**: 약 3,000여 줄의 C# 코드 작성
- **해결한 기술적 문제**: Unity 6 호환성, Live2D 다국어 지원, 안드로이드 최적화 등 15여 개 주요 이슈
- **AI 훈련 데이터**: 2,500여 줄의 프리렌 캐릭터 훈련 데이터

### 기술적 도전과 해결
#### Unity 6 호환성 문제
```csharp
// 기존 코드 (오류 발생)
KeyboardChatManager chatManager = FindObjectOfType<KeyboardChatManager>();

// 해결된 코드
KeyboardChatManager chatManager = FindFirstObjectByType<KeyboardChatManager>();
```

#### Live2D 파라미터 다국어 지원
```csharp
private string GetParameterName(string baseParam)
{
    Dictionary<string, string[]> parameterMap = new Dictionary<string, string[]>
    {
        {"AngleX", new string[] {"ParamAngleX", "角度 X", "머리_X"}},
        {"AngleY", new string[] {"ParamAngleY", "角度 Y", "머리_Y"}},
        {"EyeBlinkL", new string[] {"ParamEyeLOpen", "눈_깜빡임_L", "左眼开合"}},
        {"EyeBlinkR", new string[] {"ParamEyeROpen", "눈_깜빡임_R", "右眼开合"}}
    };
    
    if (parameterMap.ContainsKey(baseParam))
    {
        foreach (string candidate in parameterMap[baseParam])
        {
            if (cubismModel.Parameters.FindById(candidate) != null)
            {
                return candidate;
            }
        }
    }
    
    return baseParam;
}
```


## 🎯 사회적 의미 및 기여

본 프로젝트는 단순한 기술 구현을 넘어 현대 사회의 사회적 고립 문제에 대한 기술적 해법을 모색합니다. 

### 타겟 사용자
- **20-30대 정년층**: 독거노인 및 정서적 지원이 필요한 어른신
- **사회초년생**: 새로운 환경 적응에 어려움을 겪는 사람들
- **독거노인**: 사회적 고립감과 위로움을 느끼는 어르신
- **정서적 지원 필요자**: 심리적 안정을 찾는 현대인

### 기대 효과
- 가상 캐릭터를 통한 안전한 사회적 기술 연습 환경 제공
- AI 기술을 활용한 24시간 정서적 지원 서비스
- 개인화된 치료적 상호작용을 통한 심리적 안정감 증진

## 📗 참고 자료
- [Unity 6.0 Documentation](https://docs.unity3d.com/Manual/index.html)
- [Live2D Cubism SDK for Unity](https://docs.live2d.com/en/cubism-sdk-manual/unity/)
- [OpenAI Assistant API Documentation](https://platform.openai.com/docs/assistants/overview)
- [Unity 개발 문서](https://docs.google.com/document/d/1hYzwONkhU0kB6pfipcdS6Ax6zguBCO7nJdch3NEX038/edit?tab=t.0)

---

**"기술은 완벽할 필요 없다. 사람들에게 도움이 되고 의미가 있으면 된다."**

이 프로젝트를 통해 AI와 인간의 상호작용에 대한 새로운 가능성을 탐구하고, 기술을 통해 더 나은 세상을 만드는데 기여하고자 합니다.
