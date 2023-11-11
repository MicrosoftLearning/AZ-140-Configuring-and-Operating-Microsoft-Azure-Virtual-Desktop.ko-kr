---
lab:
  title: '랩: AD DS(Azure Virtual Desktop 프로필) 구현 및 관리'
  module: 'Module 4: Manage User Environments and Apps'
---

# 랩 - AD DS(Azure Virtual Desktop 프로필) 구현 및 관리
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- 완료된 랩 **WVD(AD DS)에 대한 스토리지 구현 및 관리**

## 예상 소요 시간

30분

## 실습 시나리오

AD DS(Active Directory 도메인 Services) 환경에서 Azure Virtual Desktop 프로필 관리를 구현해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop에 대한 FSLogix 기반 프로필 구현

## 랩 파일

- 없음

## 지침

### 연습 1: Azure Virtual Desktop에 대한 FSLogix 기반 프로필 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 세션 호스트 VM에서 FSLogix 기반 프로필 구성
1. Azure Virtual Desktop을 사용하여 FSLogix 기반 프로필 테스트
1. 랩에 배포된 Azure 리소스 제거

#### 작업 1: Azure Virtual Desktop 세션 호스트 VM에서 FSLogix 기반 프로필 구성

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-21-p1-0**을 선택합니다.
1. **az140-21-p1-0** 블레이드에서 **시작**을 선택하고 가상 머신의 상태가 **실행 중**으로 변경될 때까지 기다립니다.
1. **az140-21-p1-0** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-21-p1-0 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명으로 로그인합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. az140-21-p1-0**에 대한 **Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Cloud Shell 창 내에서 PowerShell 세션을 엽니다. 
1. Cloud Shell** 창의 PowerShell 세션에서 **다음을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM
   ```

   >**참고**: 다음 단계로 진행하기 전에 Azure VM이 실행될 때까지 기다립니다.

1. Cloud Shell** 창의 **PowerShell 세션에서 다음을 실행하여 세션 호스트에서 PowerShell 원격을 사용하도록 설정합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
1. Cloud Shell 닫기
1. az140-21-p1-0에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고, FSLogix 다운로드 페이지[로 이동하고](https://aka.ms/fslogix_download), FSLogix 압축 설치 이진 파일을 다운로드하고, C:\\Allfiles\\Labs\\04** 폴더로 추출**하고(필요한 경우 폴더 만들기), x64\\릴리스** 하위 폴더로 이동하고**, FSLogixAppsSetup.exe** 파일을 두 번 클릭하여 **Microsoft FSLogix 앱 설치를** 시작**합니다.** 마법사를 실행하고 기본 설정을 사용하여 Microsoft FSLogix 앱 설치를 단계별로 실행합니다.

   > **참고**: 이미지에 이미 포함된 경우 FXLogic을 설치할 필요가 없습니다.

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작하고 **관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 최신 버전의 PowerShellGet 모듈을 설치합니다(확인 메시지가 표시되면 예** 선택**).**

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
1. az140-21-p1-0에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 이 랩의 앞부분에서 구성한 Azure Storage 계정의 이름을 검색**합니다.

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName   
   ```

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 프로필 레지스트리 설정을 구성**합니다.

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   New-Item -Path $profilesParentKey -Name $profilesChildKey –Force
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
   New-ItemProperty -Path $profilesParentKey\$profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$storageAccountName.file.core.windows.net\$fileShareName"
   ```

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 시작을** 마우스 오른쪽 단추로 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 실행을 ****선택하고 **실행** 대화 상자의 **열기** 텍스트 상자에 다음을 입력하고 확인을** 선택하여 **로컬 사용자 및 그룹** 콘솔을 시작**합니다.**

   ```cmd
   lusrmgr.msc
   ```

1. **로컬 사용자 및 그룹** 콘솔에서 이름이 FSLogix** 문자열로 **시작하는 4개의 그룹을 확인합니다.

   - FSLogix ODFC Exclude List
   - FSLogix ODFC Include List
   - FSLogix Profile Exclude List
   - FSLogix Profile Include List

1. **로컬 사용자 및 그룹** 콘솔의 그룹 목록에서 **FSLogix 프로필 포함 목록** 그룹을 두 번 클릭하여 **\\Everyone** 그룹이 포함되어 있음을 확인하고 **확인**을 선택하여 그룹 **속성** 창을 닫습니다. 
1. **로컬 사용자 및 그룹** 콘솔의 그룹 목록에서 FSLogix 프로필 제외 목록** 그룹을 두 번 클릭하고 **기본적으로 그룹 구성원을 포함하지 않는다는 점에 유의하고 확인을** 선택하여 **그룹 **속성** 창을 닫습니다. 

   > **참고**: 일관된 사용자 환경을 제공하려면 모든 Azure Virtual Desktop 세션 호스트에 FSLogix 구성 요소를 설치하고 구성해야 합니다. 랩 환경의 다른 세션 호스트에서 무인 방식으로 이 작업을 수행합니다. 

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 az140-21-p1-1** 및 **az140-21-p1-2** 세션 호스트에 **FSLogix 구성 요소를 설치**합니다.

   ```powershell
   $servers = 'az140-21-p1-1', 'az140-21-p1-2'
   foreach ($server in $servers) {
      $localPath = 'C:\Allfiles\Labs\04\x64'
      $remotePath = "\\$server\C$\Allfiles\Labs\04\x64\Release"
      Copy-Item -Path $localPath\Release -Destination $remotePath -Filter '*.exe' -Force -Recurse
      Invoke-Command -ComputerName $server -ScriptBlock {
         Start-Process -FilePath $using:localPath\Release\FSLogixAppsSetup.exe -ArgumentList '/quiet' -Wait
      } 
   }
   ```

   > **참고**: 스크립트 실행이 완료되기를 기다립니다. 2분 정도 걸릴 수 있습니다.

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 az140-21-p1-1** 및 **az140-21-p1-1** 세션 호스트에서 **프로필 레지스트리 설정을 구성**합니다.

   ```powershell
   $profilesParentKey = 'HKLM:\SOFTWARE\FSLogix'
   $profilesChildKey = 'Profiles'
   $fileShareName = 'az140-22-profiles'
   foreach ($server in $servers) {
      Invoke-Command -ComputerName $server -ScriptBlock {
         New-Item -Path $using:profilesParentKey -Name $using:profilesChildKey –Force
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'Enabled' -PropertyType DWord -Value 1
         New-ItemProperty -Path $using:profilesParentKey\$using:profilesChildKey -Name 'VHDLocations' -PropertyType MultiString -Value "\\$using:storageAccountName.file.core.windows.net\$using:fileShareName"
      }
   }
   ```

   > **참고**: FSLogix 기반 프로필 기능을 테스트하기 전에 이전 랩에서 사용한 Azure Virtual Desktop 세션 호스트에서 테스트하는 데 사용할 ADATUM\\aduser1** 계정의 **로컬로 캐시된 프로필을 제거해야 합니다.

1. az140-21-p1-0에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 세션 호스트로 사용되는 모든 Azure VM에서 ADATUM\\aduser1** 계정의 **로컬로 캐시된 프로필을 제거**합니다.

   ```powershell
   $userName = 'aduser1'
   $servers = 'az140-21-p1-0','az140-21-p1-1', 'az140-21-p1-2'
   Get-CimInstance -ComputerName $servers -Class Win32_UserProfile | Where-Object { $_.LocalPath.split('\')[-1] -eq $userName } | Remove-CimInstance
   ```

#### 작업 2: Azure Virtual Desktop을 사용하여 FSLogix 기반 프로필 테스트

1. 랩 컴퓨터에서 랩 컴퓨터로 전환하고, Azure Portal을 표시하는 브라우저 창에서 가상 머신을 검색하여 선택하고 **, 가상 머신**** 블레이드에서 **az140-cl-vm11** 항목을 선택합니다**.
1. **az140-cl-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. az140-cl-vm11에 대한 **Bastion 세션 내에서 시작을 **** 클릭하고 **시작** 메뉴에서 원격 데스크톱을 클릭하여 **원격 데스크톱** 클라이언트를 시작합니다.**
1. az140-cl-vm11에 대한 **Bastion 세션 내에서 원격 데스크톱** 클라이언트 창에서 **구독**을 선택하고 **메시지가 표시되면 aduser1** 자격 증명으로 **로그인**합니다.

   >**참고** 구독하라는 메시지가 표시되지 않으면 이전 구독에서 구독을 취소해야 할 수 있습니다.
3. 애플리케이션 목록에서 **명령 프롬프트**를 두 번 클릭합니다. 그런 다음, 메시지가 표시되면 **aduser1** 계정의 암호를 입력하고 **명령 프롬프트** 창이 정상적으로 열리는지 확인합니다.
4. 명령 프롬프트 창의 **왼쪽 위 모서리에서 명령 프롬프트**** 아이콘을 **마우스 오른쪽 단추로 클릭하고 드롭다운 메뉴에서 속성을** 선택합니다**.
5. **명령 프롬프트 속성** 대화 상자에서 글꼴** 탭을 **선택하고 크기 및 글꼴 설정을 수정한 다음 확인을** 선택합니다**.
6. **명령 프롬프트** 창에서 로그오프**를 입력**하고 Enter** 키를 눌러 **원격 데스크톱 세션에서 로그아웃합니다.
7. az140-cl-vm11****에 대한 **Bastion 세션 내에서 원격 데스크톱** 클라이언트 창의 애플리케이션 목록에서 az140-21-ws1 아래의 SessionDesktop**을 두 번 클릭하고 **원격 데스크톱 세션을 시작하는지 확인합니다. 
8. **SessionDesktop** 세션 내에서 시작을** 마우스 오른쪽 단추로 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 실행을 ****선택하고 **실행** 대화 상자의 열기** 텍스트 상자에 **cmd**를 입력**하고 확인을** 선택하여 **명령 프롬프트** 창을 시작**합니다.
9. 명령 프롬프트** 창 설정이 **이 작업의 앞부분에서 구성한 설정과 일치하는지 확인합니다.
10. **SessionDesktop** 세션 내에서 모든 창을 최소화하고, 바탕 화면을 마우스 오른쪽 단추로 클릭하고, 마우스 오른쪽 단추로 클릭 메뉴에서 새로** 만들기를 선택하고**, 계단식 메뉴에서 바로 가기**를 선택합니다**. 
11. 바로 가기 만들기 마법사**의 바로 가기**를 만들려는 항목 페이지에서 **항목** 텍스트 상자의 위치를 입력하고 메모장** 입력**하고 다음**을 선택합니다**.** ** 
12. **바로 가기 만들기 마법사의 바로 가기** 페이지의 이름을 지정하려면 이 바로 가기**** 텍스트 상자의 **** 이름을 입력하고 메모장** 입력**하고 마침**을 선택합니다**.
13. **SessionDesktop** 세션 내에서 시작을 마우스 오른쪽 단추로** 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 종료 또는 로그아웃**을 선택한 **다음, 계단식 메뉴에서 로그아웃**을 선택합니다**.
14. Bastion 세션에서 **az140-cl-vm11**** 로 돌아가서 원격 데스크톱** 클라이언트 창의 애플리케이션 목록에서 SessionDesktop**을 두 번 클릭하여 **새 원격 데스크톱 세션을 시작합니다. 
15. **SessionDesktop** 세션 내에서 메모장** 바로 가기가 **바탕 화면에 표시되는지 확인합니다.
16. **SessionDesktop** 세션 내에서 시작을 마우스 오른쪽 단추로** 클릭하고 **마우스 오른쪽 단추로 클릭 메뉴에서 종료 또는 로그아웃**을 선택한 **다음, 계단식 메뉴에서 로그아웃**을 선택합니다**.
17. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 Microsoft Edge 창에서 Storage 계정** 블레이드로 **이동하고 이전 연습에서 만든 스토리지 계정을 나타내는 항목을 선택합니다.
18. 스토리지 계정 블레이드의 **파일 서비스 섹션에서 파일 공유를** 선택한 **다음 파일 공유** 목록에서 az140-22-profiles**를 선택합니다**. 
19. **az140-22-profiles** 블레이드에서 찾아보기를** 선택하고 **해당 콘텐츠에 ADATUM\\aduser1** 계정의 SID(보안 식별자) **조합과 _aduser1** 접미사로 **구성된 폴더가 포함되어 있는지 확인합니다.
20. 이전 단계에서 확인한 폴더를 선택하여 **Profile_aduser1.vhd** 파일 하나가 포함되어 있음을 확인합니다.

### 연습 2: 랩에서 프로비전되고 사용되는 Azure VM 중지 및 할당 취소

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
