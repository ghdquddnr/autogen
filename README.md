<a name="readme-top"></a>

<div align="center">
<img src="https://microsoft.github.io/autogen/0.2/img/ag.svg" alt="AutoGen Logo" width="100">

[![Twitter](https://img.shields.io/twitter/url/https/twitter.com/cloudposse.svg?style=social&label=Follow%20%40pyautogen)](https://twitter.com/pyautogen)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Company?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/company/105812540)
[![Discord](https://img.shields.io/badge/discord-chat-green?logo=discord)](https://aka.ms/autogen-discord)
[![Documentation](https://img.shields.io/badge/Documentation-AutoGen-blue?logo=read-the-docs)](https://microsoft.github.io/autogen/)
[![Blog](https://img.shields.io/badge/Blog-AutoGen-blue?logo=blogger)](https://devblogs.microsoft.com/autogen/)

</div>

<div align="center" style="background-color: rgba(255, 235, 59, 0.5); padding: 10px; border-radius: 5px; margin: 20px 0;">
  <strong>중요:</strong> 이것은 공식 프로젝트입니다. 저희는 어떤 포크나 스타트업과도 제휴하지 않습니다. 저희 <a href="https://x.com/pyautogen/status/1857264760951296210">성명서</a>를 참조하세요.
</div>

# AutoGen

**AutoGen**은 자율적으로 작동하거나 인간과 협력할 수 있는 멀티 에이전트 AI 애플리케이션을 만들기 위한 프레임워크입니다.

## 설치

AutoGen은 **Python 3.10 이상**이 필요합니다.

```bash
# Extensions에서 AgentChat과 OpenAI 클라이언트 설치
pip install -U "autogen-agentchat" "autogen-ext[openai]"
```

현재 안정 버전은 v0.4입니다. AutoGen v0.2에서 업그레이드하는 경우, 코드와 설정을 업데이트하는 방법에 대한 자세한 지침은 [마이그레이션 가이드](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/migration-guide.html)를 참조하세요.

```bash
# 노코드 GUI용 AutoGen Studio 설치
pip install -U "autogenstudio"
```

## 빠른 시작

### Hello World

OpenAI의 GPT-4o 모델을 사용하여 어시스턴트 에이전트를 생성합니다. [지원되는 다른 모델들](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/models.html)을 참조하세요.

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

async def main() -> None:
    model_client = OpenAIChatCompletionClient(model="gpt-4o")
    agent = AssistantAgent("assistant", model_client=model_client)
    print(await agent.run(task="Say 'Hello World!'"))
    await model_client.close()

asyncio.run(main())
```

### 웹 브라우징 에이전트 팀

웹 브라우징 작업을 위한 웹 서퍼 에이전트와 사용자 프록시 에이전트로 구성된 그룹 채팅 팀을 생성합니다. [playwright](https://playwright.dev/python/docs/library)를 설치해야 합니다.

```python
# pip install -U autogen-agentchat autogen-ext[openai,web-surfer]
# playwright install
import asyncio
from autogen_agentchat.agents import UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.agents.web_surfer import MultimodalWebSurfer

async def main() -> None:
    model_client = OpenAIChatCompletionClient(model="gpt-4o")
    # 웹 서퍼는 웹 브라우징 작업을 수행하기 위해 Chromium 브라우저 창을 엽니다.
    web_surfer = MultimodalWebSurfer("web_surfer", model_client, headless=False, animate_actions=True)
    # 사용자 프록시 에이전트는 웹 서퍼의 각 단계 후에 사용자 입력을 받는 데 사용됩니다.
    # 참고: Enter를 눌러 입력을 건너뛸 수 있습니다.
    user_proxy = UserProxyAgent("user_proxy")
    # 사용자가 'exit'를 입력했을 때 대화를 종료하도록 종료 조건을 설정합니다.
    termination = TextMentionTermination("exit", sources=["user_proxy"])
    # 웹 서퍼와 사용자 프록시가 라운드 로빈 방식으로 번갈아 가며 작업합니다.
    team = RoundRobinGroupChat([web_surfer, user_proxy], termination_condition=termination)
    try:
        # 팀을 시작하고 종료될 때까지 대기합니다.
        await Console(team.run_stream(task="AutoGen에 대한 정보를 찾아 간단한 요약을 작성하세요."))
    finally:
        await web_surfer.close()
        await model_client.close()

asyncio.run(main())
```

### AutoGen Studio

AutoGen Studio를 사용하여 코드 작성 없이 멀티 에이전트 워크플로우를 프로토타입하고 실행하세요.

```bash
# http://localhost:8080에서 AutoGen Studio 실행
autogenstudio ui --port 8080 --appdir ./my-app
```

## 왜 AutoGen을 사용해야 할까요?

<div align="center">
  <img src="autogen-landing.jpg" alt="AutoGen Landing" width="500">
</div>

AutoGen 생태계는 AI 에이전트, 특히 멀티 에이전트 워크플로우를 만드는 데 필요한 모든 것을 제공합니다 -- 프레임워크, 개발자 도구, 그리고 애플리케이션을.

이 _프레임워크_는 계층화되고 확장 가능한 설계를 사용합니다. 각 계층은 명확히 분리된 책임을 가지며 아래 계층 위에 구축됩니다. 이 설계를 통해 고수준 API부터 저수준 컴포넌트까지 다양한 추상화 수준에서 프레임워크를 사용할 수 있습니다.

- [Core API](./python/packages/autogen-core/)는 메시지 전달, 이벤트 기반 에이전트, 그리고 유연성과 성능을 위한 로컬 및 분산 런타임을 구현합니다. 또한 .NET과 Python의 언어 간 지원을 제공합니다.
- [AgentChat API](./python/packages/autogen-agentchat/)는 빠른 프로토타이핑을 위한 더 간단하지만 고집스러운 API를 구현합니다. 이 API는 Core API 위에 구축되며 v0.2 사용자들에게 가장 친숙하고 2-에이전트 채팅이나 그룹 채팅과 같은 일반적인 멀티 에이전트 패턴을 지원합니다.
- [Extensions API](./python/packages/autogen-ext/)는 프레임워크 기능을 지속적으로 확장하는 퍼스트파티 및 서드파티 확장을 가능하게 합니다. LLM 클라이언트(예: OpenAI, AzureOpenAI)의 특정 구현과 코드 실행과 같은 기능을 지원합니다.

생태계는 또한 두 가지 필수적인 _개발자 도구_를 지원합니다:

<div align="center">
  <img src="https://media.githubusercontent.com/media/microsoft/autogen/refs/heads/main/python/packages/autogen-studio/docs/ags_screen.png" alt="AutoGen Studio Screenshot" width="500">
</div>

- [AutoGen Studio](./python/packages/autogen-studio/)는 멀티 에이전트 애플리케이션을 구축하기 위한 노코드 GUI를 제공합니다.
- [AutoGen Bench](./python/packages/agbench/)는 에이전트 성능을 평가하기 위한 벤치마킹 도구를 제공합니다.

AutoGen 프레임워크와 개발자 도구를 사용하여 도메인에 맞는 애플리케이션을 만들 수 있습니다. 예를 들어, [Magentic-One](./python/packages/magentic-one-cli/)은 AgentChat API와 Extensions API를 사용하여 구축된 최첨단 멀티 에이전트 팀으로, 웹 브라우징, 코드 실행, 파일 처리가 필요한 다양한 작업을 처리할 수 있습니다.

AutoGen을 통해 번창하는 생태계에 참여하고 기여할 수 있습니다. 저희는 매주 유지 관리자와 커뮤니티가 함께하는 오피스 아워와 강연을 주최합니다. 또한 실시간 채팅을 위한 [Discord 서버](https://aka.ms/autogen-discord), Q&A를 위한 GitHub 토론, 그리고 튜토리얼과 업데이트를 위한 블로그가 있습니다.

## 다음은 어디로 가야 할까요?

<div align="center">

|               | [![Python](https://img.shields.io/badge/AutoGen-Python-blue?logo=python&logoColor=white)](./python)                                                                                                                                                                                                                                                                                                                | [![.NET](https://img.shields.io/badge/AutoGen-.NET-green?logo=.net&logoColor=white)](./dotnet) | [![Studio](https://img.shields.io/badge/AutoGen-Studio-purple?logo=visual-studio&logoColor=white)](./python/packages/autogen-studio)                     |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 설치  | [![Installation](https://img.shields.io/badge/Install-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/installation.html)                                                                                                                                                                                                                                                            | [![Install](https://img.shields.io/badge/Install-green)](https://microsoft.github.io/autogen/dotnet/dev/core/installation.html) | [![Install](https://img.shields.io/badge/Install-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/installation.html) |
| 빠른 시작    | [![Quickstart](https://img.shields.io/badge/Quickstart-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/quickstart.html#)                                                                                                                                                                                                                                                            | [![Quickstart](https://img.shields.io/badge/Quickstart-green)](https://microsoft.github.io/autogen/dotnet/dev/core/index.html) | [![Usage](https://img.shields.io/badge/Quickstart-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html#)        |
| 튜토리얼      | [![Tutorial](https://img.shields.io/badge/Tutorial-blue)](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/index.html)                                                                                                                                                                                                                                                            | [![Tutorial](https://img.shields.io/badge/Tutorial-green)](https://microsoft.github.io/autogen/dotnet/dev/core/tutorial.html) | [![Usage](https://img.shields.io/badge/Tutorial-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html#)        |
| API 참조 | [![API](https://img.shields.io/badge/Docs-blue)](https://microsoft.github.io/autogen/stable/reference/index.html#)                                                                                                                                                                                                                                                                                                    | [![API](https://img.shields.io/badge/Docs-green)](https://microsoft.github.io/autogen/dotnet/dev/api/Microsoft.AutoGen.Contracts.html) | [![API](https://img.shields.io/badge/Docs-purple)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/usage.html)               |
| 패키지      | [![PyPi autogen-core](https://img.shields.io/badge/PyPi-autogen--core-blue?logo=pypi)](https://pypi.org/project/autogen-core/) <br> [![PyPi autogen-agentchat](https://img.shields.io/badge/PyPi-autogen--agentchat-blue?logo=pypi)](https://pypi.org/project/autogen-agentchat/) <br> [![PyPi autogen-ext](https://img.shields.io/badge/PyPi-autogen--ext-blue?logo=pypi)](https://pypi.org/project/autogen-ext/) | [![NuGet Contracts](https://img.shields.io/badge/NuGet-Contracts-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Contracts/) <br> [![NuGet Core](https://img.shields.io/badge/NuGet-Core-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Core/) <br> [![NuGet Core.Grpc](https://img.shields.io/badge/NuGet-Core.Grpc-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.Core.Grpc/) <br> [![NuGet RuntimeGateway.Grpc](https://img.shields.io/badge/NuGet-RuntimeGateway.Grpc-green?logo=nuget)](https://www.nuget.org/packages/Microsoft.AutoGen.RuntimeGateway.Grpc/) | [![PyPi autogenstudio](https://img.shields.io/badge/PyPi-autogenstudio-purple?logo=pypi)](https://pypi.org/project/autogenstudio/)                       |

</div>


기여에 관심이 있으신가요? 시작하는 방법에 대한 가이드라인은 [CONTRIBUTING.md](./CONTRIBUTING.md)를 참조하세요. 저희는 버그 수정, 새로운 기능, 문서 개선을 포함한 모든 종류의 기여를 환영합니다. 저희 커뮤니티에 참여하여 AutoGen을 더 좋게 만들어 주세요!

질문이 있으신가요? 일반적인 질문에 대한 답변은 [자주 묻는 질문(FAQ)](./FAQ.md)을 확인하세요. 찾는 것이 없다면, [GitHub 토론](https://github.com/microsoft/autogen/discussions)에서 자유롭게 질문하거나 실시간 지원을 위해 [Discord 서버](https://aka.ms/autogen-discord)에 참여하세요. 또한 업데이트를 위한 [블로그](https://devblogs.microsoft.com/autogen/)도 읽어보실 수 있습니다.

## 법적 고지

Microsoft와 모든 기여자는 [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode) 하에 이 저장소의 Microsoft 문서 및 기타 콘텐츠에 대한 라이선스를 부여합니다. [LICENSE](LICENSE) 파일을 참조하시고, [MIT License](https://opensource.org/licenses/MIT) 하에 저장소의 모든 코드에 대한 라이선스를 부여합니다. [LICENSE-CODE](LICENSE-CODE) 파일을 참조하세요.

문서에서 참조된 Microsoft, Windows, Microsoft Azure 및/또는 기타 Microsoft 제품 및 서비스는 미국 및/또는 기타 국가에서 Microsoft의 상표이거나 등록 상표일 수 있습니다. 이 프로젝트의 라이선스는 Microsoft 이름, 로고 또는 상표를 사용할 권리를 부여하지 않습니다. Microsoft의 일반 상표 가이드라인은 <http://go.microsoft.com/fwlink/?LinkID=254653>에서 찾을 수 있습니다.

개인정보 보호 정보는 <https://go.microsoft.com/fwlink/?LinkId=521839>에서 찾을 수 있습니다.

Microsoft와 모든 기여자는 암시, 금반언 또는 기타 방법에 관계없이 각각의 저작권, 특허 또는 상표에 따른 기타 모든 권리를 보유합니다.

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
  <a href="#readme-top" style="text-decoration: none; color: blue; font-weight: bold;">
    ↑ 맨 위로 ↑
  </a>
</p>
