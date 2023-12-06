---
lab:
  title: '랩: Azure Virtual Desktop 프로필 구현 및 관리(Microsoft Entra DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# 랩 - Azure Virtual Desktop 프로필 구현 및 관리(Microsoft Entra DS)
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.
- 랩 **에서 프로비전된 Azure Virtual Desktop 환경 Azure Virtual Desktop 소개(Microsoft Entra DS)**

## 예상 소요 시간

30분

## 랩 시나리오

Microsoft Entra DS 환경에서 Azure Virtual Desktop 프로필 관리를 구현해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Entra DS 환경에서 Azure Virtual Desktop용 프로필 컨테이너를 저장하도록 Azure Files 구성
- Microsoft Entra DS 환경에서 Azure Virtual Desktop에 대한 FSLogix 기반 프로필 구현

## 랩 파일

- 없음

## 지침

### 연습 1: Azure Virtual Desktop에 대한 FSLogix 기반 프로필 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 세션 호스트 VM에서 로컬 관리istrators 그룹 구성
1. Azure Virtual Desktop 세션 호스트 VM에서 FSLogix 기반 프로필 구성
1. Azure Virtual Desktop을 사용하여 FSLogix 기반 프로필 테스트
1. Azure 랩 리소스 삭제

#### 작업 1: Azure Virtual Desktop 세션 호스트 VM에서 로컬 관리istrators 그룹 구성

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 **도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell** 창의 PowerShell 세션에서 **다음을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Start-AzVM
   ```

   >**참고**: 다음 단계로 진행하기 전에 Azure VM이 실행될 때까지 기다립니다.
   
      
1. Cloud Shell** 창의 **PowerShell 세션에서 다음을 실행하여 세션 호스트에서 PowerShell 원격을 사용하도록 설정합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21a-RG' | Enable-AzVMPSRemoting
   ```
   
1. Cloud Shell 닫기
1. 랩 컴퓨터의 Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-cl-vm11a** 항목을 선택합니다**. 그러면 az140-cl-vm11a 블레이드가 열립니 **다** .
1. **az140-cl-vm11a** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm11a \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**aadadmin1@adatum.com**|
   |암호|이전에 구성된 암호|

1. az140-cl-vm11a에 대한 **Bastion 세션 내에서 시작 메뉴 Windows 관리istration Tools** 폴더로 이동하여 **확장하고 Active Directory 사용자 및 컴퓨터** 선택합니다**.**
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 do기본 노드를 마우스 오른쪽 단추로 **클릭하고 새로 만들기를 선택한 다음 조직 구성 단위**를 선택한 **** 다음 새 **개체 - 조직 구성 단위** 대화 상자의 **이름** 텍스트 상자에 ADDC 사용자를** 입력**하고 확인을** 선택합니다**.
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 ADDC 사용자를** 마우스 오른쪽 단추로 클릭하고 **새로** 만들기를 선택한 **다음 **그룹** 뒤에 있는 새 개체 - 그룹** 대화 상자에서 **다음 설정을 지정하고 확인을** 선택합니다**.

   |설정|값|
   |---|---|
   |그룹 이름|**로컬 관리자**|
   |그룹 이름(Windows 2000 이전)|**로컬 관리자**|
   |Group scope|**Global**|
   |그룹 형식|**보안**|

1. **Active Directory 사용자 및 컴퓨터** 콘솔에 **로컬 관리자** 그룹의 속성을 표시하고 **구성원** 탭으로 전환하여 **추가**를 선택합니다. 그런 다음, **사용자, 연락처, 컴퓨터, 서비스 계정 또는 그룹 선택** 대화 상자의 **선택할 개체 이름 입력**에 **aadadmin1;wvdaadmin1**을 입력하고 **확인**을 선택합니다.
1. az140-cl-vm11a**에 대한 **Bastion 세션 내에서 시작 메뉴 Windows 관리istration Tools** 폴더로 이동하여 **확장한 다음 그룹 정책 관리를** 선택합니다**.
1. **그룹 정책 관리** 콘솔에서 AADDC 컴퓨터 OU로 이동하고 **AADDC 컴퓨터** GPO 아이콘을 마우스 오른쪽 단추로** 클릭하고 **편집**을 선택합니다**.
1. **그룹 정책 관리 편집기** 콘솔에서 컴퓨터 구성 **, 정책 **, ****Windows 설정**, **보안 설정** 확장하고**, 나머지 그룹을** 마우스 오른쪽 단추로 클릭하고 **그룹 추가**를 선택합니다**.
1. 그룹 추가 대화 상자의 **그룹** 텍스트 상자에서 **찾아보기를** 선택하고 **그룹 **** 선택 대화 상자의 선택할 개체 이름** 입력에서 **로컬 관리** 입력**하고 확인을** 선택합니다**.**
1. 그룹 추가 대화 상자로 **돌아가서 확인을** 선택합니다**.**
1. **ADATUM\Local 관리s 속성** 대화 상자의 이 그룹이 구성원**이라는 레이블이 지정된 **섹션에서 추가**를 선택하고 **그룹 멤버 자격** 대화 상자에서 **관리istrators**를 입력**하고 확인을** 선택한 다음 확인을 다시 선택하여 ****** 변경 내용을 완료합니다.

   >**참고**: 이 그룹이 의 구성원이라는 레이블이 지정된 **섹션을 사용해야 합니다.**

1. az140-cl-vm11a Azure VM에 대한 Bastion 세션 내에서 PowerShell ISE를 관리istrator로 시작하고 다음을 실행하여 그룹 정책 처리를 트리거하기 위해 두 Azure Virtual Desktop 호스트를 다시 시작합니다.

   ```powershell
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Restart-Computer -ComputerName $servers -Force -Wait
   ```

1. 스크립트가 완료될 때까지 기다립니다. 이 작업은 3분 정도 걸립니다.

#### 작업 2: Azure Virtual Desktop 세션 호스트 VM에서 FSLogix 기반 프로필 구성

1. az140-cl-vm11a에 대한 **Bastion 세션 내에서 원격 데스크톱 세션을 **az140-21-p1-0**으로 시작하고 메시지가 표시되면 ADATUM\wvdaadmin1** 사용자 이름과 이 사용자 계정을 만들 때 설정한 암호로 **로그인합니다.** 

   > **참고**: RDP 연결을 연결할 수 없는 경우 Azure Portal을 사용하여 Bastion을 사용하여 VM에 연결합니다.

1. **az140-21-p1-0**에 연결된 원격 데스크톱 세션 내에서 Microsoft Edge를 시작하고 [FSLogix 다운로드 페이지](https://aka.ms/fslogix_download)로 이동합니다. 그런 다음, 압축된 FSLogix 설치 이진 파일을 다운로드하여 **C:\\Source** 폴더에 압축을 풉니다. 그 후에 **x64\\Release** 하위 폴더로 이동한 다음, **FSLogixAppsSetup.exe**를 사용하여 기본 설정으로 Microsoft FSLogix Apps를 설치합니다.

   > **참고**: 이미지에 이미 포함되어 있는지 여부에 따라 FXLogic 설치가 필요하지 않을 수 있습니다. FX Logix를 설치하려면 다시 부팅해야 합니다.

1. az140-21-p1-0에 대한 **원격 데스크톱 세션 내에서 관리자 권한으로 Windows PowerShell ISE**를 시작하고 **관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 최신 버전의 PowerShellGet 모듈을 설치합니다(확인 메시지가 표시되면 예** 선택**).**

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 최신 버전의 Az PowerShell 모듈을 설치합니다(확인 메시지가 표시되면 모두 예** 선택**).

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 실행 정책을 수정합니다.

   ```powershell
   Set-ExecutionPolicy RemoteSigned -Force
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독에 로그인합니다.

   ```powershell
   Connect-AzAccount
   ```

1. 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할을 사용하여 사용자 계정의 Microsoft Entra 자격 증명으로 로그인합니다.
1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩의 앞부분에서 구성한 Azure Storage 계정의 이름을 검색합니다.

   ```powershell
   $resourceGroupName = 'az140-22a-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. az140-21-p1-0에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 프로필 레지스트리 설정을 구성**합니다.

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```
   >**참고** 명령에서 오류가 발생하면 다음 단계를 계속하세요.
   
1. az140-21-p1-0에 대한 **원격 데스크톱 세션 내에서 시작을** 마우스 오른쪽 단추로 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 실행을**** 선택하고 **실행** 대화 상자의 열기** 텍스트 상자에 **다음을 입력하고 확인을** 선택하여 **로컬 사용자 및 그룹** 창을 시작**합니다.**

   ```cmd
   lusrmgr.msc
   ```

1. **로컬 사용자 및 그룹** 콘솔에서 이름이 FSLogix** 문자열로 **시작하는 4개의 그룹을 확인합니다.

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. **로컬 사용자 및 그룹** 콘솔에서 **FSLogix 프로필 포함 목록** 그룹 항목을 두 번 클릭하고 여기에 **\\Everyone** 그룹이 포함되어 있는지 확인한 다음, **확인**을 선택하여 그룹 **속성** 창을 닫습니다. 
1. **로컬 사용자 및 그룹** 콘솔에서 FSLogix 프로필 제외 목록** 그룹 엔티를 두 번 클릭하고 **기본적으로 그룹 구성원을 포함하지 않는다는 점에 유의하고 확인을** 선택하여 **그룹 **속성** 창을 닫습니다. 

   > **참고**: 일관된 사용자 환경을 제공하려면 모든 Azure Virtual Desktop 세션 호스트에 FSLogix 구성 요소를 설치하고 구성해야 합니다. 랩 환경의 다른 세션 호스트에서 무인 방식으로 이 작업을 수행합니다. 

1. az140-21-p1-0에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 az140-21-p1-1** 세션 호스트에 **FSLogix 구성 요소를 설치**합니다.

   ```powershell
   $server = 'az140-21-p1-1' 
   $localPath = 'C:\Source\x64'
   $remotePath = "\\$server\C$\Source\x64\Release"
   Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
   Invoke-Command -ComputerName $server -ScriptBlock {
      Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
   } 
   ```

1. az140-21-p1-0에 대한 **원격 데스크톱 세션 내에서 관리자 권한으로 Windows PowerShell ISE**를 시작하고 **관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 az140-21-p1-1** 세션 호스트에서 **프로필 레지스트리 설정을 구성**합니다.

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22a-profiles'
   Invoke-Command -ComputerName $server -ScriptBlock {
      New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
      New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$using:fileShareName"
   }
   ```

   > **참고**: FSLogix 기반 프로필 기능을 테스트하기 전에 이전 랩에서 사용한 Azure Virtual Desktop 세션 호스트에서 테스트하는 데 사용할 ADATUM\wvdaadmin1 계정의 로컬로 캐시된 프로필을 제거해야 합니다.

1. Bastion 세션**으로 az140-cl-vm11a**로 전환하고, Bastion 세션 **내에서 az140-cl-vm11a**로 전환하고, 관리istrator: Windows PowerShell ISE** 창으로 전환**하고, 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 ADATUM\aaduser1 계정의 로컬로 캐시된 프로필을 제거합니다.

   ```powershell
   $userName = 'aaduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### 작업 3: Azure Virtual Desktop을 사용하여 FSLogix 기반 프로필 테스트

1. az140-cl-vm11a**에 대한 **Bastion 세션 내에서 원격 데스크톱 클라이언트로 전환합니다.
1. az140-cl-vm11a에 대한 **Bastion 세션 내에서 원격 데스크톱** 클라이언트 창의 애플리케이션 목록에서 명령 프롬프트**를 두 번 클릭하고 **메시지가 표시되면 암호를 제공하고 명령 프롬프트** 창을 시작하는지 확인합니다**.**** 

   > **참고**: 처음에는 애플리케이션을 시작하는 데 몇 분 정도 걸릴 수 있지만 이후에는 애플리케이션 시작 속도가 훨씬 빨라야 합니다.

1. 명령 프롬프트 창의 **왼쪽 위 모서리에서 명령 프롬프트**** 아이콘을 **마우스 오른쪽 단추로 클릭하고 드롭다운 메뉴에서 속성을** 선택합니다**.
1. **명령 프롬프트 속성** 대화 상자에서 글꼴** 탭을 **선택하고 크기 및 글꼴 설정을 수정한 다음 확인을** 선택합니다**.
1. **명령 프롬프트** 창에서 로그오프**를 입력**하고 Enter** 키를 눌러 **원격 데스크톱 세션에서 로그아웃합니다.
1. az140-cl-vm11a****에 대한 **Bastion 세션 내에서 원격 데스크톱** 클라이언트 창의 애플리케이션 목록에서 SessionDesktop**을 두 번 클릭하고 **원격 데스크톱 세션을 시작하는지 확인합니다. 
1. **SessionDesktop** 세션 내에서 시작을** 마우스 오른쪽 단추로 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 실행을 ****선택하고 **실행** 대화 상자의 열기** 텍스트 상자에 **cmd**를 입력**하고 확인을** 선택하여 **명령 프롬프트** 창을 시작**합니다.
1. 명령 프롬프트** 창 속성이 **이 작업의 앞부분에서 설정한 속성과 일치하는지 확인합니다.
1. **SessionDesktop** 세션 내에서 모든 창을 최소화하고, 바탕 화면을 마우스 오른쪽 단추로 클릭하고, 마우스 오른쪽 단추로 클릭 메뉴에서 새로** 만들기를 선택하고**, 계단식 메뉴에서 바로 가기**를 선택합니다**. 
1. 바로 가기 만들기 마법사**의 바로 가기**를 만들려는 항목 페이지에서 **항목** 텍스트 상자의 위치를 입력하고 메모장** 입력**하고 다음**을 선택합니다**.** ** 
1. **바로 가기 만들기 마법사의 바로 가기** 페이지의 이름을 지정하려면 이 바로 가기**** 텍스트 상자의 **** 이름을 입력하고 메모장** 입력**하고 마침**을 선택합니다**.
1. **SessionDesktop** 세션 내에서 시작을 마우스 오른쪽 단추로** 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 종료 또는 로그아웃**을 선택한 **다음, 계단식 메뉴에서 로그아웃**을 선택합니다**.
1. Bastion 세션에서 az140-cl-vm11a**로 **돌아가서 원격 데스크톱** 클라이언트 창의 애플리케이션 목록에서 SessionDesktop**을 두 번 클릭하여 **새 원격 데스크톱 세션을 시작**합니다. 
1. **SessionDesktop** 세션 내에서 메모장** 바로 가기가 **바탕 화면에 표시되는지 확인합니다.
1. **SessionDesktop** 세션 내에서 시작을 마우스 오른쪽 단추로** 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 종료 또는 로그아웃**을 선택한 **다음, 계단식 메뉴에서 로그아웃**을 선택합니다**.
1. Bastion 세션 **으로 az140-cl-vm11a**로 전환하고 Azure Portal을 표시하는 Microsoft Edge 창으로 전환합니다.
1. Azure Portal을 표시하는 Microsoft Edge 창에서 Storage 계정** 블레이드로 **돌아가서 이전 연습에서 만든 스토리지 계정을 나타내는 항목을 선택합니다.
1. 스토리지 계정 블레이드의 **파일 서비스 섹션에서 파일 공유를** 선택한 **다음 파일 공유** 목록에서 az140-22a-profiles**를 선택합니다**. 
1. **az140-22a-profiles** 블레이드에서 찾아보기를** 선택하고 **해당 콘텐츠에 ADATUM\\aaduser1** 계정의 SID(보안 식별자) **조합과 _aaduser1** 접미사로 **구성된 폴더가 포함되어 있는지 확인합니다.
1. 이전 단계에서 식별한 폴더를 선택하고 Profile_aaduser1.vhd**라는 **단일 파일이 포함되어 있음을 확인합니다.

### 연습 2: Azure 랩 리소스 삭제(선택 사항)

1. Azure[ Portal을 사용하여 Azure Active Directory 도메인 Services 관리 do기본 삭제에 ]( https://docs.microsoft.com/en-us/azure/active-directory-domain-services/delete-aadds)설명된 지침에 따라 Microsoft Entra DS 배포를 제거합니다.
1. Azure Resource Manager 리소스 그룹 및 리소스 삭제[에 설명된 지침에 따라 이 과정의 Microsoft Entra DS 랩에서 ](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-portal)프로비전한 모든 Azure 리소스 그룹을 제거합니다.
