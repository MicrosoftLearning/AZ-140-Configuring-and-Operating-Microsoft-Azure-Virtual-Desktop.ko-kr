---
lab:
  title: '랩: AVD(AD DS)에 대한 조건부 액세스 정책 구성'
  module: 'Module 3: Manage Access and Security'
---

# 랩 - AVD(AD DS)에 대한 조건부 액세스 정책 구성
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- **Azure Portal을 사용하여 호스트 풀 및 세션 호스트 배포(AD DS)** 랩 완료

## 예상 소요 시간

60분

## 랩 시나리오

Microsoft Entra 조건부 액세스를 사용하여 AD DS(Active Directory Domain Services) 환경에서 Azure Virtual Desktop 배포에 대한 액세스를 제어해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 준비
- Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 구현

## 랩 파일

- None 

## 지침

>**중요**: Microsoft는 **Azure Active Directory**(**Azure AD**)의 이름을 **Microsoft Entra ID**로 변경했습니다. 이 변경 내용에 대한 자세한 내용은 [Azure Active Directory의 새 이름](https://learn.microsoft.com/en-us/entra/fundamentals/new-name)을 참조하세요. 이는 지속적인 활동이므로 개별 연습을 진행하면서 랩 지침과 인터페이스 요소가 일치하지 않는 경우가 여전히 발생할 수 있습니다. 다음 사항을 고려합니다. 특히 이 랩에서는, **Microsoft Entra Connect**는 **Azure Active Directory Connect**라는 새 이름을 지정하며, 연습 1의 작업 4에서 서비스 연결점을 구성할 때 **Azure Active Directory**라는 용어가 계속 사용됩니다.

>**중요**: Microsoft Entra ID P2 평가판을 활성화하려면 신용 카드 정보를 제공해야 합니다. 이러한 이유로 이 연습은 전적으로 선택 사항입니다. 대신, 과정 강사는 학생에게 이 기능을 시연하도록 선택할 수 있습니다.

### 연습 1: Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 준비

이 연습의 주요 작업은 다음과 같습니다.

1. Microsoft Entra Premium P2 라이선스 구성
1. Microsoft Entra MFA(Multi-Factor Authentication) 구성
1. Microsoft Entra MFA 사용자 등록
1. 하이브리드 Microsoft Entra 조인 구성
1. Microsoft Azure Active Directory Connect 델타 동기화 트리거

#### 작업 1: Microsoft Entra Premium P2 라이선스 구성

>**참고**: Microsoft Entra 조건부 액세스를 구현하려면 Microsoft Entra의 프리미엄 P1 또는 P2 라이선스가 필요합니다. 이 랩에서는 30일 평가판을 사용합니다.

1. 랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure Portal](https://portal.azure.com)로 이동한 후 이 랩에서 사용할 구독의 소유자 역할과 해당 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 제공하여 로그인합니다.

    >**중요**: 회사 또는 학교 계정(Microsoft 계정이 **아님**)을 사용하고 있는지 확인합니다.

1. Azure Portal에서 **Azure Active Directory**를 검색 및 선택하여 이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Azure Active Directory 블레이드 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **사용자**를 클릭합니다. 
1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 **aduser5**를 선택합니다.
1. **aduser5 | 프로필** 블레이드의 도구 모음에서 **편집**을 클릭합니다. 그런 다음, **설정** 섹션의 **사용 위치** 드롭다운 목록에서 랩 환경이 있는 국가를 선택하고 도구 모음에서 **저장**을 클릭합니다.
1. **aduser5 | 프로필** 블레이드의 **ID** 섹션에서 **aduser5** 계정의 사용자 계정 이름을 확인합니다.

    >**참고**: 이 값을 기록해 둡니다. 이 랩의 후반부에서 필요합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 이 작업을 시작할 때 로그인할 때 사용했던 사용자 계정을 선택합니다. 계정에 **사용 위치**가 할당되어 있지 않으면 이전 단계를 반복합니다. 

    >**참고**: Microsoft Entra Premium P2 라이선스를 사용자 계정에 할당하려면 **사용 위치** 속성을 설정해야 합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 **aadsyncuser** 사용자 계정을 선택하여 해당 사용자 계정 이름을 확인합니다.

    >**참고**: 이 값을 기록해 둡니다. 이 랩의 후반부에서 필요합니다.

1. Azure Portal에서 Microsoft Entra 테넌트의 **개요** 블레이드로 돌아가서 왼쪽 세로 메뉴 모음의 **관리** 섹션에서 **라이선스**를 클릭합니다.
1. **라이선스 \| 개요** 블레이드 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **모든 제품**을 클릭합니다.
1. **라이선스 \| 모든 제품** 블레이드의 도구 모음에서 **+ 사용/구매**를 클릭합니다.
1. **활성화** 블레이드의 **MICROSOFT ENTRA ID P2** 섹션에서 **평가판**을 클릭한 다음 **활성화**를 클릭하고 안내에 따라 활성화 프로세스를 완료합니다.
1. **라이선스 - 모든 제품** 블레이드에서 **Enterprise Mobility + Security E5** 항목을 선택합니다. 
1. **Enterprise Mobility + Security E5** 블레이드의 도구 모음에서 **+ 할당**을 선택합니다.
1. **라이선스 할당** 블레이드에서 **사용자 및 그룹 추가**를 클릭하고 **사용자 및 그룹 추가** 블레이드에서 **aduser5** 및 자신의 사용자 계정을 선택한 후에 **선택**을 클릭합니다.
1. **라이선스 할당** 블레이드에서 **할당 옵션**을 클릭하고 **할당 옵션** 블레이드에서 모든 옵션이 사용하도록 설정되어 있는지 확인한 후에 **검토 + 할당**, **할당**을 차례로 클릭합니다.

#### 작업 2: Microsoft Entra MFA(Multi-Factor Authentication) 구성

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 Microsoft Entra 테넌트의 **개요 **블레이드로 다시 이동한 후 왼쪽 세로 메뉴에 있는 **관리** 섹션에서 **보안**을 클릭합니다.
1. **보안 | 시작** 블레이드 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **ID 보호**를 클릭합니다.
1. **Identity Protection | 개요** 창 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **다단계 인증 등록 정책**을 클릭합니다(필요한 경우 웹 브라우저 페이지를 새로 고침).
1. **ID 보호 | 다단계 인증 등록 정책** 창에서 **다단계 인증 등록 정책**의 **할당** 섹션에서 **모든 사용자**를 클릭합니다. 그런 다음, **포함** 탭에서 **개인 및 그룹 선택** 옵션을 클릭하고 **사용자 선택**에서 **aduser5**를 클릭한 후에 **선택**을 클릭합니다. 그 후에 창 아래쪽에서 **정책 적용** 스위치를 **켜기**로 설정하고 **저장**을 클릭합니다.

#### 작업 3: Microsoft Entra MFA 사용자 등록

1. 랩 컴퓨터에서 **InPrivate** 웹 브라우저 세션을 열고 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 연습 앞부분에서 확인한 **aduser5** 사용자 계정 이름과 이 사용자 계정을 만들 때 설정한 암호를 입력하여 로그인합니다.
1. **자세한 정보 필요** 메시지가 표시되면 **다음**을 클릭합니다. 그러면 브라우저가 **Microsoft Authenticator** 페이지로 자동 리디렉션됩니다.
1. **추가 보안 인증** 페이지의 **1단계: 어떻게 연락해야 하나요?** 에서 원하는 인증 방법을 선택하고 지침을 따라 등록 프로세스를 완료합니다. 
1. Azure Portal 페이지 오른쪽 위에서 사용자 아바타에 해당하는 아이콘을 클릭하고 **로그아웃**을 클릭한 후에 **In private** 브라우저 창을 닫습니다. 

#### 작업 4: 하이브리드 Microsoft Entra 조인 구성

> **참고**: 이 기능은 Microsoft Entra 조인 상태에 따라 디바이스에 대한 조건부 액세스를 설정할 때 추가 보안을 구현하는 데 활용될 수 있습니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-dc-vm11**을 선택합니다.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **Bastion을 통해 연결**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. **az140-dc-vm11**에 연결된 Bastion 세션 내의 **시작** 메뉴에서 **Azure AD Connect** 폴더를 확장하고 **Azure AD Connect**를 선택합니다.

   > **참고** 동기화 서비스가 실행되고 있지 않다는 실패 오류 창이 나타나면 PowerShell 명령 창으로 이동하여 **Start-Service "ADSync"** 를 입력한 다음, 이전 단계를 다시 시도합니다.

1. **Microsoft Azure Active Directory Connect** 창의 **Azure AD Connect 시작** 페이지에서 **구성**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **추가 작업** 페이지에서 **디바이스 옵션 구성**을 선택하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **개요** 페이지에서 **하이브리드 Microsoft Entra 조인** 및 **디바이스 쓰기 저장**에 관한 정보를 검토하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **Microsoft Entra에 연결** 페이지에서 이전 랩에서 만든 **aadsyncuser** 사용자 계정의 자격 증명을 사용하여 인증하고 **다음**을 선택합니다.  
1. **Microsoft Azure Active Directory Connect** 창의 **디바이스 옵션** 페이지에서 **하이브리드 Azure AD 조인 구성** 옵션이 선택되어 있는지 확인하고 **다음**을 선택합니다. 
1. **Microsoft Azure Active Directory Connect** 창의 **디바이스 운영 체제** 페이지에서 **Windows 10 이상 도메인 조인 디바이스** 체크박스를 선택하고 **다음**을 선택합니다. 
1. **Microsoft Azure Active Directory Connect** 창의 **SCP 구성** 페이지에서 **adatum.com** 항목 옆의 체크박스를 선택합니다. 그런 다음 **인증 서비스** 드롭다운 목록에서 **Azure Active Directory** 항목을 선택하고 **추가**를 선택합니다. 
1. 메시지가 표시되면 **엔터프라이즈 관리자 자격 증명** 대화 상자에서 다음 자격 증명을 지정하고 **확인**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\Student**|
   |암호|**Pa55w.rd1234**|

1. **Microsoft Azure Active Directory Connect** 창의 **SCP 구성** 페이지로 돌아와서 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **구성 준비 완료** 페이지에서 **구성**을 선택하고 구성이 완료되면 **끝내기**를 선택합니다.
1. **az140-dc-vm11**에 연결된 Bastion 세션 내에서 관리자로 **Windows PowerShell ISE**를 시작합니다.
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 **az140-cl-vm11** 컴퓨터 계정을 **WVDClients** OU(조직 구성 단위)로 이동합니다.

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. **az140-dc-vm11**에 연결된 Bastion 세션 내의 **시작** 메뉴에서 **Azure AD Connect** 폴더를 확장하고 **Azure AD Connect**를 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **Azure AD Connect 시작** 페이지에서 **구성**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **추가 작업** 페이지에서 **동기화 옵션 사용자 지정**을 선택하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **Microsoft Entra에 연결** 페이지에서 이전 연습에서 만든 **aadsyncuser** 사용자 계정의 자격 증명을 사용하여 인증하고 **다음**을 선택합니다. 
1. **Microsoft Azure Active Directory Connect** 창의 **디렉터리 연결** 페이지에서 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **도메인 및 OU 필터링** 페이지에서 **선택한 도메인 및 OU 동기화** 옵션이 선택되어 있는지 확인합니다. 그런 다음 **adatum.com** 노드를 확장하여 **ToSync** OU 옆의 체크박스가 선택되어 있는지 확인하고 **WVDClients** OU 옆의 체크박스를 선택한 후 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **선택적 기능** 페이지에서 기본 설정을 적용하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **구성 준비 완료** 페이지에서 **구성이 완료되면 동기화 프로세스를 시작합니다.** 체크박스가 선택되었는지 확인하고 **구성**을 선택합니다.
1. **구성 완료** 페이지의 정보를 검토하고 **끝내기**를 선택하여 **Microsoft Azure Active Directory Connect** 창을 닫습니다.

#### 작업 5: Microsoft Azure Active Directory Connect 전체 동기화 트리거

1. 랩 컴퓨터에 표시된 Azure Portal에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-cl-vm11a** 항목을 선택합니다. 그러면 **az140-cl-vm11** 블레이드가 열립니다.
1. **az140-cl-vm11** 블레이드 내에서 **다시 시작**을 선택한 다음 **가상 머신을 성공적으로 다시 시작함** 알림이 나타날 때까지 기다립니다.
1. **az140-dc-vm11**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 창으로 전환합니다.
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell ISE** 콘솔 창에서 다음을 실행하여 Microsoft Azure Active Directory Connect 전체 동기화를 트리거합니다.

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. **az140-dc-vm11**에 연결된 Bastion 세션 내에서 Microsoft Edge를 시작하고 [Azure Portal](https://portal.azure.com)로 이동합니다. 메시지가 표시되면 이 랩에서 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. **az140-dc-vm11**에 연결된 Bastion 세션 내의 Azure Portal을 표시하는 Microsoft Edge 창에서 **Microsoft Entra ID**를 검색하고 선택하여 이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Microsoft Entra ID 블레이드 왼쪽 세로 메뉴 모음의 **관리** 섹션에서 **디바이스**를 클릭합니다. 
1. **디바이스 | 모든 디바이스** 블레이드에서 디바이스 목록을 검토하고 **az140-cl-vm11** 디바이스가 **조인 형식** 열의 **Microsoft Entra hybrid Join** 항목과 함께 나열되어 있는지 확인합니다.

   > **참고**: Azure Portal에 디바이스가 표시되기 전에 동기화가 적용되기까지 몇 분 기다려야 할 수도 있습니다.

### 연습 2: Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 구현

이 연습의 주요 작업은 다음과 같습니다.

1. 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 만들기
1. 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 테스트
1. Microsoft Entra 기반 조건부 액세스 정책을 수정하여 MFA 요구 사항에서 하이브리드 Microsoft Entra 조인 컴퓨터 제외
1. 수정된 Microsoft Entra 기반 조건부 액세스 정책 테스트

#### 작업 1: 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 만들기

>**참고**: 이 작업에서는 MFA가 Azure Virtual Desktop 세션에 로그인하도록 요구하는 Microsoft Entra 기반 조건부 액세스 정책을 구성합니다. 또한 이 정책은 정상 인증 후 첫 4시간이 지나면 재인증을 강제 적용합니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 Microsoft Entra 테넌트의 **개요 **블레이드로 다시 이동한 후 왼쪽 세로 메뉴에 있는 **관리** 섹션에서 **보안**을 클릭합니다.
1. **보안 \| 시작** 블레이드 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **조건부 액세스**를 클릭합니다.
1. **조건부 액세스 \| 정책** 창의 도구 모음에서 **+ 새 정책**을 클릭합니다.
1. **새로 만들기** 블레이드에서 다음 설정을 구성합니다.

   - **이름** 텍스트 상자에 **az140-31-wvdpolicy1**을 입력합니다.
   - **할당** 섹션에서 **사용자 또는 워크로드 ID** 옵션을 선택하고 **이 정책은 무엇에 적용되나요?** 드롭다운 목록에서 ** 사용자 및 그룹**을 선택하고 **사용자 및 그룹 선택** 섹션에서 **사용자 및 그룹** 확인란을 선택하고 **선택** 블레이드에서 **aduser5**를 클릭한 다음, **선택**을 클릭합니다.
   - **할당** 섹션에서 **클라우드 앱 또는 작업**을 클릭하고 **이 정책을 적용할 항목 선택** 스위치에서 **클라우드 앱** 옵션이 선택되어 있는지 확인합니다. 그런 다음, **앱 선택** 옵션을 클릭하고 **선택** 창의 **검색** 텍스트 상자에 **Azure Virtual Desktop**을 입력합니다. 그 후 결과 목록에서 **Azure Virtual Desktop** 항목 옆에 있는 체크박스를 선택하고 **검색** 텍스트 상자에 **Microsoft Remote Desktop**를 입력한 다음, **Microsoft 원격 데스크톱** 항목 옆에 있는 체크박스를 선택하고 **선택**을 클릭합니다. 

   > **참고**: Azure Virtual Desktop(앱 ID 9cdead84-a844-4324-93f2-b2e6bb768d07)은 사용자가 피드를 구독하고 연결 중에 Azure Virtual Desktop 게이트웨이에 인증할 때 사용됩니다. Microsoft 원격 데스크톱(앱 ID a4a365df-50f1-4397-bc59-1a1564b8bb9c)은 Single Sign-On이 사용하도록 설정된 경우 사용자가 세션 호스트에 인증할 때 사용됩니다.

   - **할당** 섹션에서 **조건**을 클릭하고 **클라이언트 앱**을 클릭합니다. 그런 다음 **클라이언트 앱** 블레이드에서 **구성** 스위치를 **예**로 설정하고 **브라우저**와 **모바일 앱 및 데스크톱 클라이언트** 체크박스가 모두 선택되어 있는지 확인한 후 **완료**를 클릭합니다.
   - **액세스 제어** 섹션에서 **권한 부여**를 클릭하고 **권한 부여** 블레이드에서 **액세스 허용** 옵션이 선택되었는지 확인합니다. 그런 다음 **다단계 인증 필요** 체크박스를 선택하고 **선택**을 클릭합니다.
   - **액세스 제어** 섹션에서 **세션**을 클릭하고 **세션** 블레이드에서 **로그인 빈도** 체크박스를 선택합니다. 첫 번째 텍스트 상자에는 **4**를 입력하고, **단위 선택** 드롭다운 목록에서 **시간**을 선택합니다. **영구 브라우저 세션** 체크박스는 선택하지 않은 상태로 유지하고 **선택**을 클릭합니다.
   - **정책 사용** 스위치를 **켜기**로 설정합니다.

1. **새로 만들기** 블레이드에서 **만들기**를 클릭합니다. 

#### 작업 2: 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 테스트

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저 창에서 **Cloud Shell** 창 내에 **PowerShell** 셸 세션을 엽니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**참고**: 명령이 완료되고 **az140-21-RG** 리소스 그룹에 있는 모든 Azure VM이 실행될 때까지 기다립니다. 

1. 랩 컴퓨터에서 **InPrivate** 웹 브라우저 세션을 열고 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 연습 앞부분에서 확인한 **aduser5** 사용자 계정 이름과 이 사용자 계정을 만들 때 설정한 암호를 입력하여 로그인합니다.

   > **참고**: MFA를 통해 인증하라는 메시지가 표시되지 않음을 확인합니다.

1. **InPrivate** 웹 브라우저 Azure Virtual Desktop HTML5 웹 클라이언트 페이지 [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)로 이동합니다.

   > **참고**: 이 페이지로 이동하면 MFA를 통한 인증이 자동으로 트리거되는지 확인합니다.

1. **코드 입력** 창에서 등록한 문자 메시지 또는 인증 앱의 코드를 입력하고 **확인**을 선택합니다.
1. **모든 리소스** 페이지에서 **명령 프롬프트**를 클릭하고 **로컬 리소스 액세스** 창에서 **프린터** 체크박스 선택을 취소한 후에 **허용**을 클릭합니다.
1. 메시지가 표시되면 **자격 증명 입력**, **사용자 이름** 텍스트 상자에 **aduser5**의 사용자 계정 이름을 입력하고 **암호** 입력란에 텍스트 상자에 이 사용자 계정을 만들 때 설정한 암호를 입력하고 **제출**을 클릭합니다.
1. **명령 프롬프트** 원격 앱이 정상적으로 시작되었는지 확인합니다.
1. **명령 프롬프트** 원격 앱 창의 명령 프롬프트에 **logoff**를 입력하고 **Enter** 키를 누릅니다.
1. **모든 리소스** 페이지로 돌아와 오른쪽 위에서 **aduser5**를 클릭하고 드롭다운 메뉴에서 **로그아웃**을 클릭한 다음 **InPrivate** 웹 브라우저 창을 닫습니다.

#### 작업 3: Microsoft Entra 기반 조건부 액세스 정책을 수정하여 MFA 요구 사항에서 하이브리드 Microsoft Entra 조인 컴퓨터 제외

>**참고**: 이 작업에서는 Azure Virtual Desktop 세션에 로그인하려면 MFA를 사용해야 하는 Microsoft Entra 기반 조건부 액세스 정책을 수정하여 Microsoft Entra 조인 컴퓨터에서 시작되는 연결에는 MFA를 사용하지 않아도 되도록 설정합니다.

1. 랩 컴퓨터에서 Azure Portal을 표시하는 브라우저 창의 **조건부 액세스 | 정책** 블레이드에서 **az140-31-wvdpolicy1** 정책을 나타내는 항목을 클릭합니다.
1. **az140-31-wvdpolicy1** 블레이드의 **액세스 제어** 섹션에서 **권한 부여**를 클릭하고 **권한 부여** 블레이드에서 **다단계 인증 필요** 및 **하이브리드 Microsoft Entra 조인 디바이스 필요** 체크박스를 선택합니다. 그런 후에 **선택한 컨트롤 중 하나가 필요함** 옵션이 사용하도록 설정되어 있는지 확인하고 **선택**을 클릭합니다.
1. **az140-31-wvdpolicy1** 블레이드에서 **저장**을 클릭합니다.

>**참고**: 정책을 적용하는 데 몇 분 정도 걸릴 수 있습니다.

#### 작업 4: 수정된 Microsoft Entra 기반 조건부 액세스 정책 테스트

1. 랩 컴퓨터의 Azure Portal이 표시된 브라우저 창에서 **가상 머신**을 검색하여 선택합니다. 그런 다음 **가상 머신** 블레이드에서 **az140-cl-vm11** 항목을 선택합니다.
1. **az140-cl-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. **az140-cl-vm11**에 연결된 Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Virtual Desktop HTML5 웹 클라이언트 페이지 [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)로 이동합니다.

   > **참고**: 이번에는 MFA를 통해 인증하라는 메시지가 표시되지 않음을 확인합니다. 이는 **az140-cl-vm11**이 Hybrid Microsoft Entra에 조인되어 있기 때문입니다.

1. **모든 리소스** 페이지에서 **명령 프롬프트**를 두 번 클릭하고 **로컬 리소스 액세스** 창에서 **프린터** 체크박스 선택을 취소한 후에 **허용**을 클릭합니다.
1. 메시지가 표시되면 **자격 증명 입력**, **사용자 이름** 텍스트 상자에 **aduser5**의 사용자 계정 이름을 입력하고 **암호** 입력란에 텍스트 상자에 이 사용자 계정을 만들 때 설정한 암호를 입력하고 **제출**을 클릭합니다.
1. **명령 프롬프트** 원격 앱이 정상적으로 시작되었는지 확인합니다.
1. **명령 프롬프트** 원격 앱 창의 명령 프롬프트에 **logoff**를 입력하고 **Enter** 키를 누릅니다.
1. **모든 리소스** 페이지로 돌아와 오른쪽 위에서 **aduser5**를 클릭하고 드롭다운 메뉴에서 **로그아웃**을 클릭합니다.
1. **az140-cl-vm11**에 연결된 Bastion 세션 내에서 **시작**을 클릭합니다. 그런 다음 **시작** 단추 바로 위의 세로 막대에서 로그인한 사용자 계정에 해당하는 아이콘을 클릭하고 팝업 메뉴에서 **로그아웃**을 클릭합니다.

### 연습 3: 랩에서 프로비전 및 사용한 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전 및 사용한 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 해당 컴퓨팅 비용을 최소화하기 위해 이 랩에서 프로비전 및 사용한 Azure VM의 할당을 취소합니다.

#### 작업 1: 랩에서 프로비전 및 사용한 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 **Cloud Shell** 창 내에서 **PowerShell** 셸 세션을 엽니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM의 목록을 표시합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 비동기적으로 실행되므로(-NoWait 매개 변수에 의해 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수는 있지만, Azure VM이 실제로 중지 및 할당 취소되려면 몇 분 정도 걸립니다.
