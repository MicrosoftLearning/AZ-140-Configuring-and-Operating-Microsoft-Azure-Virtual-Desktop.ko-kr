---
lab:
  title: '랩: 세션 호스트(Entra ID)에 연결'
  module: 'Module 3.2: Plan and implement user experience and client settings'
---

# 랩 - 세션 호스트(Entra ID)에 연결
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독
- 이 랩에서 사용할 Azure 구독에 대한 Owner 또는 Contributor 역할을 가지며 해당 Azure 구독에 연결된 Entra 테넌트에 디바이스를 가입시기에 충분한 사용 권한이 있는 Microsoft Entra 사용자 계정
- *Azure Portal(Entra ID)을 사용하여 호스트 풀 및 세션 호스트 배포* 랩을 완료
- *Azure Portal(Entra ID)을 사용하여 호스트 풀 및 세션 호스트 관리* 랩을 완료

## 예상 소요 시간

20분

## 랩 시나리오

Entra 조인 세션 호스트를 포함하는 기존 Azure Virtual Desktop 환경이 있습니다. Microsoft Entra에 조인되거나 등록되지 않은 Windows 11 클라이언트에서 이 호스트에 연결하여 호스트 기능의 유효성을 검사해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Entra에 조인되거나 등록되지 않은 Windows 클라이언트에서 Microsoft Entra 조인 Azure Virtual Desktop 세션 호스트에 연결하여 호스트 기능의 유효성을 검사합니다.

## 랩 파일

- None

## 지침

### 연습 1: Windows 11 클라이언트에서 Microsoft Entra 조인 Azure Virtual Desktop 세션 호스트에 연결하여 기능 유효성 검사
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 호스트 풀의 RDP 속성 조정
1. Windows 11 컴퓨터에 Microsoft 원격 데스크톱 클라이언트 설치
1. Azure Virtual Desktop 작업 영역 구독
1. Azure Virtual Desktop 앱 테스트


#### 작업 1: Azure Virtual Desktop 호스트 풀의 RDP 속성 조정

> **참고**: 이전 랩에서 구현한 RDP 설정은 (Single Sign-On 지원을 통해) 최적의 사용자 환경을 제공하지만 [Microsoft Entra ID 인증을 사용하여 Azure Virtual Desktop용 Single Sign-On 구성](https://learn.microsoft.com/en-us/azure/virtual-desktop/configure-single-sign-on)에서 설명하는 추가적인 변경이 필요합니다. 이렇게 변경하지 않는 경우 기본적으로 클라이언트 컴퓨터가 다음 조건 중 하나를 충족할 때 인증이 지원됩니다.

- 세션 호스트와 동일한 Microsoft Entra 테넌트에 Microsoft Entra 조인된 경우
- 세션 호스트와 동일한 Microsoft Entra 테넌트에 Microsoft Entra 하이브리드 조인된 경우
- 세션 호스트와 동일한 Microsoft Entra 테넌트에 Microsoft Entra 등록된 경우

랩 컴퓨터에는 위 조건이 적용되지 않으므로 `targetisaadjoined:i:1`을 사용자 지정 RDP 속성으로 호스트 풀에 추가해야 합니다.

1. 필요한 경우 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal로 이동하여 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

    > **참고**: 랩 세션 창의 오른쪽에 있는 리소스 탭에 나열된 `User1-` 계정의 자격 증명을 사용합니다.

1. Azure Portal을 표시하는 웹 브라우저에서 **az140-21-hp1** 페이지의 세로 메뉴 모음**의 설정** 섹션에서 **RDP 속성** 항목을 선택합니다.
1. **az140-21-hp1 \| RDP 속성** 페이지에서 **고급** 탭을 선택합니다. 
1. **az140-21-hp1 \| RDP 속성** 페이지의 **고급** 탭에 있는 **RDP 속성** 텍스트 상자에서 기존 콘텐츠에 다음 문자열을 추가합니다(이때, 앞의 문자열과 구분하려면 선행 세미콜론 문자(`;`)를 추가해야 합니다).

    ```txt
    targetisaadjoined:i:1
    ```

1. **RDP 속성** 텍스트 상자의 기존 콘텐츠에서 다음 문자열(있는 경우)을 제거합니다(후행 세미콜론 문자 포함).

    ```txt
    enablerdsaadauth:i:value
    ```

1. **az140-21-hp1 \| RDP 속성** 페이지에서 **저장**을 선택합니다.

#### 작업 2: Windows 11 컴퓨터에 Microsoft 원격 데스크톱 클라이언트 설치

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Windows용 원격 데스크톱 클라이언트를 사용하여 Azure Virtual Desktop에 연결](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows) 페이지로 이동하고, 아래로 스크롤하여 **원격 데스크톱 클라이언트(MSI) 다운로드 및 설치** 섹션에서 [Windows 64비트](https://go.microsoft.com/fwlink/?linkid=2139369) 링크를 선택합니다. 
1. 파일 탐색기 열고 **다운로드** 폴더로 이동한 다음 새로 다운로드한 MSI 파일의 설치를 시작합니다. 
1. **원격 데스크톱 설정** 창에서 메시지가 표시되면 라이선스 계약 조건을 수락하고 **이 컴퓨터의 모든 사용자에 대해 설치** 옵션을 선택합니다. 메시지가 표시되면 사용자 계정 컨트롤 메시지를 수락하여 설치를 진행합니다.
1. 설치가 완료되면 **설치가 종료되면 원격 데스크톱 시작** 확인란이 선택되어 있는지 확인하고, **마침**을 선택하여 Microsoft 원격 데스크톱 클라이언트를 시작합니다.

   > **참고**: Windows용 [원격 데스크톱 스토어 앱](https://learn.microsoft.com/en-us/azure/virtual-desktop/users/connect-windows?pivots=rd-store)은 Microsoft Entra 조인 세션 호스트에 대한 연결을 지원하지 않습니다.

#### 작업 3: Azure Virtual Desktop 작업 영역 구독

1. 랩 컴퓨터에서 **원격 데스크톱** 클라이언트 창으로 전환하여 **구독**을 선택하고, 메시지가 표시되면 랩 인터페이스 창의 오른쪽 창에 있는 **리소스** 탭에 표시된 `User1` Entra ID 사용자 계정의 자격 증명으로 로그인합니다.

   > **참고**: **AVD-DAG** 접두사를 가진 Entra 그룹의 구성원인 사용자 계정을 선택합니다.

   > **참고**: 다른 방법으로는 **원격 데스크톱** 클라이언트 창에서 **URL로 구독**을 선택하고 **작업 영역 구독** 창의 **이메일 또는 작업 영역 URL**에 **https://client.wvd.microsoft.com/api/arm/feeddiscovery**를 입력하고 **다음**을 선택합니다. 그런 다음 메시지가 표시되면 Microsoft Entra 자격 증명으로 로그인합니다.

1. **원격 데스크톱** 페이지에 **SessionDesktop** 아이콘만 표시되는지 확인합니다.

   > **참고**: 이는 예상된 일로, 선택한 Microsoft Entra 사용자 계정이 첫 번째 랩 *Azure Portal(Entra ID)을 사용하여 호스트 풀 및 세션 호스트 배포*에서 자동 생성된 **az140-21-hp1-DAG** 데스크톱 애플리케이션 그룹에 할당되었기 때문입니다.

1. **원격 데스크톱** 페이지에서 **SessionDesktop** 아이콘을 마우스 오른쪽 단추로 클릭하고 팝업 메뉴에서 **설정**을 선택합니다.
1. **SessionDesktop** 창에서 **기본 설정 사용**스위치를 끕니다.
1. **디스플레이 설정** 섹션의 드롭다운 메뉴에서 **디스플레이 선택** 항목을 선택하고 세션에 사용할 디스플레이를 선택합니다.
1. **SessionDesktop** 창에서 **현재 디스플레이에 최대화**, **창 모드에서 단일 디스플레이**, **창에 세션 맞춤**을 비롯한 나머지 옵션을 검토하되 변경하지 않습니다. 
1. **SessionDesktop** 창을 닫습니다. 
1. **원격 데스크톱** 페이지에서 **SessionDesktop** 아이콘을 두 번 클릭합니다.
1. 로그인하라는 메시지가 표시되면 **Windows 보안** 대화 상자에서 이 작업에서 대상 Azure Virtual Desktop 환경에 연결하는 데 사용한 첫 번째 Microsoft Entra 사용자 계정의 암호를 입력합니다.

   > **참고**: Azure Virtual Desktop은 하나의 사용자 계정으로 Microsoft Entra ID에 로그인한 다음, 별도의 사용자 계정으로 Windows에 로그인하는 것을 지원하지 않습니다. 두 개의 서로 다른 계정으로 동시에 로그인하면 사용자가 잘못된 세션 호스트에 다시 연결하고, Azure Portal에서 정보가 잘못되거나 누락되고, 앱 연결 또는 MSIX 앱 연결을 사용하는 동안 오류 메시지가 표시될 수 있습니다.

   > **참고**: **SessionDesktop** 창이 자동으로 표시됩니다.

1. 원격 데스크톱 세션 창에서 세션 내에서 전체 관리 엑세스 권한이 있는지 확인합니다(예: 작업 표시줄에서 **Windows** 로고 아이콘을 선택한 다음 팝업 메뉴에서 **Windows PowerShell(관리자)** 항목을 선택합니다.
1. 원격 데스크톱 세션 창에서 작업 표시줄에서 Windows 로고 아이콘을 선택하고, 로그인하는 데 사용한 Microsoft Entra 사용자 계정을 나타내는 아바타 아이콘을 선택하고, 팝업 메뉴에서 **로그아웃**을 선택합니다.

   > **참고**: 이 작업은 원격 데스크톱 세션을 자동으로 종료시킵니다. 

1. **원격 데스크톱** 창으로 돌아가 **az140-21-ws1** 작업 영역 항목의 오른쪽에 있는 줄임표(`...`) 아이콘을 선택하고, **구독 취소**를 선택한 다음 확인 메시지가 표시되면 **계속**을 선택합니다.
1. **원격 데스크톱** 클라이언트 창에서 **구독**을 선택하고 메시지가 표시되면 랩 인터페이스 창의 오른쪽 창에 있는 **리소스** 탭에서 찾을 수 있는 두 번째 Entra ID 사용자 계정의 자격 증명으로 로그인합니다.

   > **참고**: **AVD-RemoteApp** 접두사가 있는 Entra 그룹의 구성원인 사용자 계정을 선택합니다.

1. **원격 데스크톱** 페이지에 명령 프롬프트, Microsoft Word, Microsoft Excel, Microsoft PowerPoint를 포함한 4개의 아이콘이 표시되는지 확인합니다. 

   > **참고**: 이는 예상되는 것으로, 선택한 Microsoft Entra 사용자 계정이 첫 번째 랩 *Azure Portal(Entra ID)을 사용하여 호스트 풀 및 세션 호스트 배포*에서 **az140-21-hp1-Office365-RAG** 및 **az140-21-hp1-Utilities-RAG** 애플리케이션 그룹에 할당되었기 때문입니다.

1. 명령 프롬프트 아이콘을 두 번 클릭합니다. 
1. 로그인하라는 메시지가 표시되면 **Windows 보안** 대화 상자에서 대상 Azure Virtual Desktop 환경에 연결하는 데 사용한 두 번째 Microsoft Entra 사용자 계정의 암호를 입력합니다.
1. **명령 프롬프트** 창이 곧 표시되는지 확인합니다. 
1. 명령 프롬프트에서 **호스트 이름**을 입력하고 **Enter** 키를 눌러 명령 프롬프트가 실행 중인 컴퓨터의 이름을 표시합니다.

   > **참고**: 표시된 이름이 **sh-** 접두사로 시작하는지 확인합니다.

1. 명령 프롬프트에 **로그오프**를 입력하고 **Enter** 키를 눌러 현재 원격 앱 세션에서 로그오프합니다.
1. **원격 데스크톱** 페이지에서 나머지 아이콘을 두 번 클릭하여 Microsoft Word, Microsoft Excel 및 Microsoft PowerPoint를 시작합니다.
1. 각 세션 창을 닫습니다.
