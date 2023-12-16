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
- 완료된 랩 **은 AD DS(Azure Portal)를 사용하여 호스트 풀 및 세션 호스트를 배포합니다.**

## 예상 소요 시간

60분

## 랩 시나리오

Microsoft Entra 조건부 액세스를 사용하여 AD DS(Active Directory 도메인 Services) 환경에서 Azure Virtual Desktop 배포에 대한 액세스를 제어해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 준비
- Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 구현

## 랩 파일

- 없음 

## 지침

### 연습 1: Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 준비

이 연습의 주요 작업은 다음과 같습니다.

1. Microsoft Entra Premium P2 라이선스 구성
1. Microsoft Entra MFA(Multi-Factor Authentication) 구성
1. Microsoft Entra MFA에 대한 사용자 등록
1. 하이브리드 Microsoft Entra 조인 구성
1. Microsoft Entra 커넥트 델타 동기화 트리거

#### 작업 1: Microsoft Entra Premium P2 라이선스 구성

>**참고**: Microsoft Entra 조건부 액세스를 구현하려면 Microsoft Entra의 프리미엄 P1 또는 P2 라이선스가 필요합니다. 이 랩에 대해 30일 평가판을 사용합니다.

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독의 소유자 역할과 해당 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할을 사용하여 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 Azure Active Directory**를 검색하여 선택하여 **이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Azure Active Directory 블레이드의 왼쪽**에 있는 세로 메뉴 모음의 관리** 섹션에서 사용자를** 클릭합니다**. 
1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 **aduser5**를 선택합니다.
1. **aduser5 | 프로필** 블레이드의 도구 모음에서 **편집**을 클릭합니다. 그런 다음, **설정** 섹션의 **사용 위치** 드롭다운 목록에서 랩 환경이 있는 국가를 선택하고 도구 모음에서 **저장**을 클릭합니다.
1. **aduser5 | 프로필** 블레이드의 **ID** 섹션에서 **aduser5** 계정의 사용자 계정 이름을 확인합니다.

    >**참고**: 이 값을 기록해 둡니다. 이 랩의 후반부에서 필요합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 이 작업을 시작할 때 로그인할 때 사용했던 사용자 계정을 선택합니다. 계정에 **사용 위치**가 할당되어 있지 않으면 이전 단계를 반복합니다. 

    >**참고**: **사용자 계정에 Microsoft Entra Premium P2 라이선스를 할당하려면 사용 위치** 속성을 설정해야 합니다.

1. **사용자 | 모든 사용자(미리 보기)** 블레이드에서 **aadsyncuser** 사용자 계정을 선택하여 해당 사용자 계정 이름을 확인합니다.

    >**참고**: 이 값을 기록해 둡니다. 이 랩의 후반부에서 필요합니다.

1. Azure Portal에서 Microsoft Entra 테넌트 개요** 블레이드로 **다시 이동하고 왼쪽**의 세로 메뉴 모음에서 관리** 섹션에서 라이선스**를 클릭합니다**.
1. **라이선스 \| 개요** 블레이드 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **모든 제품**을 클릭합니다.
1. **라이선스 \| 모든 제품** 블레이드의 도구 모음에서 **+ 사용/구매**를 클릭합니다.
1. 활성화 블레이드의 ENTERPRISE MOBILITY + SECURITY E5** 섹션에서 평가판을** **클릭한 다음 활성화**를 클릭합니다**.****** 
1. **라이선스 \| 개요** 블레이드에서 브라우저 창을 새로 고쳐 활성화가 정상적으로 완료되었는지 확인합니다. 
1. **라이선스 - 모든 제품 블레이드에서** Enterprise Mobility + Security E5** 항목을 선택합니다**. 
1. **Enterprise Mobility + Security E5** 블레이드의 도구 모음에서 + 할당**을 클릭합니다**.
1. **라이선스 할당** 블레이드에서 **사용자 및 그룹 추가**를 클릭하고 **사용자 및 그룹 추가** 블레이드에서 **aduser5** 및 자신의 사용자 계정을 선택한 후에 **선택**을 클릭합니다.
1. 라이선스 할당 블레이드로 **돌아가서 할당 옵션을 클릭하고 **할당 옵션** 블레이드에서 **모든 옵션이** 사용하도록 설정되어 있는지 확인하고 검토 + 할당을 클릭하고 **할당****을 클릭합니다**.**

#### 작업 2: Microsoft Entra MFA(Multi-Factor Authentication) 구성

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 Microsoft Entra 테넌트 개요 블레이드로 **돌아가 왼쪽의 세로 메뉴에서 **[관리**] 섹션에서 [보안 **]을 클릭합니다**.**
1. **보안 | 시작** 블레이드 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **ID 보호**를 클릭합니다.
1. **Identity Protection | 개요** 창 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **다단계 인증 등록 정책**을 클릭합니다(필요한 경우 웹 브라우저 페이지를 새로 고침).
1. **ID 보호 | 다단계 인증 등록 정책** 창에서 **다단계 인증 등록 정책**의 **할당** 섹션에서 **모든 사용자**를 클릭합니다. 그런 다음, **포함** 탭에서 **개인 및 그룹 선택** 옵션을 클릭하고 **사용자 선택**에서 **aduser5**를 클릭한 후에 **선택**을 클릭합니다. 그 후에 창 아래쪽에서 **정책 적용** 스위치를 **켜기**로 설정하고 **저장**을 클릭합니다.

#### 작업 3: Microsoft Entra MFA에 대한 사용자 등록

1. 랩 컴퓨터에서 **InPrivate** 웹 브라우저 세션을 열고 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 연습 앞부분에서 확인한 **aduser5** 사용자 계정 이름과 이 사용자 계정을 만들 때 설정한 암호를 입력하여 로그인합니다.
1. 필요한** 추가 정보가 포함된 메시지가 **표시되면 다음**을 클릭합니다**. 그러면 브라우저가 Microsoft Authenticator** 페이지로 **자동으로 리디렉션됩니다.
1. **추가 보안 확인** 페이지의 **1단계: 어떻게 연락해야 하나요?** 섹션에서 기본 인증 방법을 선택하고 지침에 따라 등록 프로세스를 완료합니다. 
1. Azure Portal 페이지의 오른쪽 위 모서리에서 사용자 아바타를 나타내는 아이콘을 클릭하고 로그아웃을 클릭한 **다음 프라이빗** 브라우저에서 창을 닫습니다**.** 

#### 작업 4: 하이브리드 Microsoft Entra 조인 구성

> **참고**: 이 기능을 활용하여 Microsoft Entra 조인 상태 기반으로 디바이스에 대한 조건부 액세스를 설정할 때 추가 보안을 구현할 수 있습니다.

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 시작** 메뉴에서 Microsoft Entra 커넥트** 폴더를 확장하고 **Microsoft Entra 커넥트** 선택합니다**.****
   > **참고** 동기화 서비스가 실행되고 있지 않다는 실패 오류 창이 나타나면 PowerShell 명령 창으로 이동하여 **Start-Service “ADSync”** 를 입력한 다음, 4단계를 다시 시도합니다.
1. **Microsoft Entra 커넥트** 창의 **Microsoft Entra 커넥트** 시작 페이지에서 구성**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **추가 작업** 페이지에서 디바이스 옵션** 구성을 선택하고 **다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **개요** 페이지에서 하이브리드 Microsoft Entra 조**인 및 디바이스 쓰기 저장**에 **대한 **정보를 검토하고 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트 창의 Microsoft Entra** 커넥트** 페이지에서 이전 연습에서 **만든 aadsyncuser** 사용자 계정의 **자격 증명을 사용하여 인증하고 다음**을 선택합니다**.  

   > **참고**: 이 랩에서 이전에 기록한 aadsyncuser** 계정의 **userPrincipalName 특성을 제공하고 이 사용자 계정을 만들 때 설정한 암호를 지정합니다. 

1. **Microsoft Entra 커넥트** 창의 **디바이스 옵션** 페이지에서 하이브리드 Microsoft Entra 조**인 구성 옵션이 선택되어 있는지 확인하고 **다음**을 선택합니다**. 
1. **Microsoft Entra 커넥트** 창**의 **디바이스 운영 체제** 페이지에서 Windows 10 이상 조인 디바이스** 검사box에서 기본 다음을 선택합니다****. 
1. **Microsoft Entra 커넥트 창의 **SCP 구성** 페이지에서 adatum.com** 항목 옆에 **있는 검사** 상자를 선택하고 인증 **서비스** 드롭다운 목록에서 Microsoft Entra** 항목을 선택하고 **추가**를 선택합니다**. 
1. 메시지가 표시되면 Enterprise 관리 자격 증명** 대화 상자에서 **다음 자격 증명을 지정하고 확인을** 선택합니다**.

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\Student**|
   |암호|**Pa55w.rd1234**|

1. **Microsoft Entra 커넥트** 창의 **SCP 구성** 페이지로 돌아가서 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **구성** 준비 페이지에서 구성**을 선택하고 **구성이 완료되면 종료**를 선택합니다**.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작**** 합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 az140-cl-vm11** 컴퓨터 계정을 **WVDClients** OU(조직 구성 단위)로 이동합니다**.**

   ```powershell
   Move-ADObject -Identity "CN=az140-cl-vm11,CN=Computers,DC=adatum,DC=com" -TargetPath "OU=WVDClients,DC=adatum,DC=com"
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 시작** 메뉴에서 Microsoft Entra 커넥트** 폴더를 확장하고 **Microsoft Entra 커넥트** 선택합니다**.****
1. **Microsoft Entra 커넥트** 창의 **Microsoft Entra 커넥트** 시작 페이지에서 구성**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **추가 작업** 페이지에서 동기화 옵션** 사용자 지정을 선택하고 **다음**을 선택합니다**.
1. **Microsoft Entra 커넥트 창의 Microsoft Entra** 커넥트** 페이지에서 이전 연습에서 **만든 aadsyncuser** 사용자 계정의 **자격 증명을 사용하여 인증하고 다음**을 선택합니다**. 

   > **참고**: 이 랩에서 이전에 기록한 aadsyncuser** 계정의 **userPrincipalName 특성을 제공하고 이 사용자 계정을 만들 때 설정한 암호를 지정합니다. 

1. **Microsoft Entra 커넥트 창의 **디렉터리** 커넥트** 페이지에서 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 Do기본 및 OU 필터링** 페이지에서 **선택한 do기본 및 OU** 옵션이 **선택되어 있는지 확인하고, adatum.com** 노드를 확장하고**, ToSync** OU 옆에 **있는 검사 상자가 선택되어 있는지 확인하고, WVDClients** OU 옆에 있는 **검사 상자를 선택하고, 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **선택적 기능** 페이지에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **구성** 준비 페이지에서 구성이 완료되면** 검사box **동기화 프로세스를 시작하고 구성**을 선택합니다**.
1. 구성 완료** 페이지에서 정보를 검토하고 종료**를 **선택하여 **Microsoft Entra 커넥트** 창을 닫습니다**.

#### 작업 5: Microsoft Entra 커넥트 델타 동기화 트리거

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 창으로 전환**** 합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔 창에서 **다음을 실행하여 Microsoft Entra 커넥트 델타 동기화를 트리거**합니다.

   ```powershell
   Import-Module -Name "C:\Program Files\Microsoft Azure AD Sync\Bin\ADSync"
   Start-ADSyncSyncCycle -PolicyType Initial
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할과 함께 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. az140-dc-vm11**에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Active Directory**를 검색하고 선택하여 **이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Azure Active Directory 블레이드의 왼쪽**에 있는 세로 메뉴 모음의 관리** 섹션에서 디바이스**를 클릭합니다**. 
1. 디바이스 |** 모든 디바이스** 블레이드에서 디바이스 목록을 검토하고 az140-cl-vm11** 디바이스가 조인 유형** 열에 **하이브리드 Microsoft Entra 조인 항목과 함께 **나열되는지** 확인**합니다.

   > **참고**: 디바이스가 Azure Portal에 표시되기 전에 동기화가 수행될 때까지 몇 분 정도 기다려야 할 수 있습니다.

### 연습 2: Azure Virtual Desktop에 대한 Microsoft Entra 기반 조건부 액세스 구현

이 연습의 주요 작업은 다음과 같습니다.

1. 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 만들기
1. 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 테스트
1. MFA 요구 사항에서 하이브리드 Microsoft Entra 조인 컴퓨터를 제외하도록 Microsoft Entra 기반 조건부 액세스 정책 수정
1. 수정된 Microsoft Entra 기반 조건부 액세스 정책 테스트

#### 작업 1: 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 만들기

>**참고**: 이 작업에서는 MFA가 Azure Virtual Desktop 세션에 로그인해야 하는 Microsoft Entra 기반 조건부 액세스 정책을 구성합니다. 또한 이 정책은 인증에 성공한 후 처음 4시간 후에 다시 인증을 적용합니다.

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 Microsoft Entra 테넌트 개요 블레이드로 **돌아가 왼쪽의 세로 메뉴에서 **[관리**] 섹션에서 [보안 **]을 클릭합니다**.**
1. **보안 \| 시작** 블레이드 왼쪽의 세로 메뉴에 있는 **보호** 섹션에서 **조건부 액세스**를 클릭합니다.
1. **조건부 액세스 \| 정책** 창의 도구 모음에서 **+ 새 정책**을 클릭합니다.
1. **새로 만들기** 블레이드에서 다음 설정을 구성합니다.

   - **이름** 텍스트 상자에 az140-31-wvdpolicy1을 입력**합니다.**
   - **할당** 섹션에서 **사용자 또는 워크로드 ID** 옵션을 선택하고 **이 정책은 무엇에 적용되나요?** 드롭다운 목록에서 ** 사용자 및 그룹**을 선택하고 **사용자 및 그룹 선택** 섹션에서 **사용자 및 그룹** 확인란을 선택하고 **선택** 블레이드에서 **aduser5**를 클릭한 다음, **선택**을 클릭합니다.
   - **할당** 섹션에서 **클라우드 앱 또는 작업**을 클릭하고 **이 정책을 적용할 항목 선택** 스위치에서 **클라우드 앱** 옵션이 선택되어 있는지 확인합니다. 그런 다음, **앱 선택** 옵션을 클릭하고 **선택** 창의 **검색** 텍스트 상자에 **9cdead84-a844-4324-93f2-b2e6bb768d07**을 입력합니다. 그 후 결과 목록에서 **Azure Virtual Desktop** 항목 옆에 있는 체크박스를 선택하고 **검색** 텍스트 상자에 **a4a365df-50f1-4397-bc59-1a1564b8bb9c**를 입력한 다음, **Microsoft 원격 데스크톱** 항목 옆에 있는 체크박스를 선택하고 **선택**을 클릭합니다. 

   > **참고**: Azure Virtual Desktop(앱 ID 9cdead84-a844-4324-93f2-b2e6bb768d07)은 사용자가 피드를 구독하고 연결 중에 Azure Virtual Desktop 게이트웨이에 인증할 때 사용됩니다. Microsoft 원격 데스크톱(앱 ID a4a365df-50f1-4397-bc59-1a1564b8bb9c)은 Single Sign-On이 사용하도록 설정된 경우 사용자가 세션 호스트에 인증할 때 사용됩니다.

   - 할당 섹션에서 조건을** 클릭하고 **클라이언트 앱을** 클릭하고 **클라이언트 앱** 블레이드에서 **구성** 스위치를 예**로 **설정하고 **브라우저** 및 모바일 앱과 **데스크톱 클라이언트** 검사 상자가 모두 **선택되었는지 확인하고 완료**를 클릭합니다**.** ** 
   - **액세스 제어** 섹션에서 권한 부여**를 클릭하고**, 권한 부여** 블레이드에서 **액세스 권한** 부여 옵션이 선택되어 있는지 확인하고**, 다단계 인증** 필요 검사 상자를 선택하고 **선택을** 클릭합니다**.
   - **Access 컨트롤** 섹션에서 세션을** 클릭하고 **세션** **블레이드에서 로그인 빈도** 검사 상자를 선택하고**, 첫 번째 텍스트 상자에 4**를 입력**하고, **단위** 선택 드롭다운 목록에서 시간을** 선택하고**, 영구 브라우저 세션** 검사 상자를 지운 상태로 두고**, 선택을** 클릭합니다**.
   - 정책** 사용 스위치를 **켜**기로 **설정합니다.

1. **새로 만들기** 블레이드에서 **만들기**를 클릭합니다. 

#### 작업 2: 모든 Azure Virtual Desktop 연결에 대한 Microsoft Entra 기반 조건부 액세스 정책 테스트

1. 랩 컴퓨터와 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**참고**: 명령이 완료되고 az140-21-RG** 리소스 그룹의 모든 Azure VM**이 실행될 때까지 기다립니다. 

1. 랩 컴퓨터에서 **InPrivate** 웹 브라우저 세션을 열고 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 연습 앞부분에서 확인한 **aduser5** 사용자 계정 이름과 이 사용자 계정을 만들 때 설정한 암호를 입력하여 로그인합니다.

   > **참고**: MFA를 통해 인증하라는 메시지가 표시되지 않는지 확인합니다.

1. **InPrivate** 웹 브라우저 Azure Virtual Desktop HTML5 웹 클라이언트 페이지 [https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)로 이동합니다.

   > **참고**: MFA를 통해 인증이 자동으로 트리거되는지 확인합니다.

1. **코드 입력** 창에서 등록한 문자 메시지 또는 인증 앱의 코드를 입력하고 **확인**을 선택합니다.
1. **모든 리소스** 페이지에서 **명령 프롬프트**를 클릭하고 **로컬 리소스 액세스** 창에서 **프린터** 체크박스 선택을 취소한 후에 **허용**을 클릭합니다.
1. 메시지가 표시되면 **자격 증명 입력**, **사용자 이름** 텍스트 상자에 **aduser5**의 사용자 계정 이름을 입력하고 **암호** 입력란에 텍스트 상자에 이 사용자 계정을 만들 때 설정한 암호를 입력하고 **제출**을 클릭합니다.
1. **명령 프롬프트** 원격 앱이 정상적으로 시작되었는지 확인합니다.
1. **명령 프롬프트** 원격 앱 창의 명령 프롬프트에 **logoff**를 입력하고 **Enter** 키를 누릅니다.
1. **모든 리소스** 페이지로 돌아가서 오른쪽 위 모서리에서 aduser5**를 클릭하고 **드롭다운 메뉴에서 로그아웃**을 클릭하고 **InPrivate** 웹 브라우저 창을 닫습니다**.

#### 작업 3: MFA 요구 사항에서 하이브리드 Microsoft Entra 조인 컴퓨터를 제외하도록 Microsoft Entra 기반 조건부 액세스 정책 수정

>**참고**: 이 작업에서는 MFA가 Azure Virtual Desktop 세션에 로그인해야 하는 Microsoft Entra 기반 조건부 액세스 정책을 수정하여 Microsoft Entra 조인 컴퓨터에서 시작된 연결에 MFA가 필요하지 않도록 합니다.

1. 랩 컴퓨터에서 Azure Portal을 표시하는 브라우저 창의 **조건부 액세스 | 정책** 블레이드에서 **az140-31-wvdpolicy1** 정책을 나타내는 항목을 클릭합니다.
1. **az140-31-wvdpolicy1** 블레이드의 **Access Controls** 섹션에서 권한 부여**를 클릭하고**, 권한 부여** 블레이드에서 **다단계 인증** 필요 및 **하이브리드 Microsoft Entra 조인 디바이스** 검사Boxes를 선택하고**, 선택한 컨트롤** 옵션 중 하나가 사용하도록 설정되어 있는지 확인하고**, 선택을** 클릭합니다**.
1. **az140-31-wvdpolicy1** 블레이드에서 저장**을 클릭합니다**.

>**참고**: 정책이 적용되는 데 몇 분 정도 걸릴 수 있습니다.

#### 작업 4: 수정된 Microsoft Entra 기반 조건부 액세스 정책 테스트

1. 랩 컴퓨터의 Azure Portal을 표시하는 브라우저 창에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-cl-vm11** 항목을 선택합니다**.
1. **az140-cl-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. az140-cl-vm11에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Virtual Desktop HTML5 웹 클라이언트 페이지[https://rdweb.wvd.microsoft.com/arm/webclient](https://rdweb.wvd.microsoft.com/arm/webclient)로 이동합니다.**

   > **참고**: 이번에는 MFA를 통해 인증하라는 메시지가 표시되지 않는지 확인합니다. az140-cl-vm11**이 하이브리드 Microsoft Entra 조인이기 때문**입니다.

1. **모든 리소스** 페이지에서 **명령 프롬프트**를 두 번 클릭하고 **로컬 리소스 액세스** 창에서 **프린터** 체크박스 선택을 취소한 후에 **허용**을 클릭합니다.
1. 메시지가 표시되면 **자격 증명 입력**, **사용자 이름** 텍스트 상자에 **aduser5**의 사용자 계정 이름을 입력하고 **암호** 입력란에 텍스트 상자에 이 사용자 계정을 만들 때 설정한 암호를 입력하고 **제출**을 클릭합니다.
1. **명령 프롬프트** 원격 앱이 정상적으로 시작되었는지 확인합니다.
1. **명령 프롬프트** 원격 앱 창의 명령 프롬프트에 **logoff**를 입력하고 **Enter** 키를 누릅니다.
1. 모든 리소스** 페이지로 돌아가서 오른쪽 위 모서리에서 aduser5**를 클릭하고 **드롭다운 메뉴에서 로그아웃**을 클릭합니다**.**
1. az140-cl-vm11에 대한 **Bastion 세션 내에서 시작을** 클릭하고 **시작** 단추 바로 위의 **세로 막대에서 로그인한 사용자 계정을 나타내는 아이콘을 클릭한 다음 팝업 메뉴에서 로그아웃**을 클릭합니다**.**

### 연습 3: 랩에서 프로비전되고 사용되는 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전되고 사용되는 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에서 프로비전되고 사용되는 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전되고 사용되는 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만들고 사용하는 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
