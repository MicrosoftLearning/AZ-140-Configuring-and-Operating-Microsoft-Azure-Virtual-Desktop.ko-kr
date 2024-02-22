---
lab:
  title: '랩: AD DS(Azure Virtual Desktop 애플리케이션) 패키지'
  module: 'Module 4: Manage User Environments and Apps'
---

# 랩 - AD DS(Azure Virtual Desktop 애플리케이션) 패키지
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- 완료된 랩 **AD DS(Azure Virtual Desktop 프로필 관리)**
- 완료된 랩 **WVD에 대한 조건부 액세스 정책 구성(AD DS)**

## 예상 소요 시간

60분

## 랩 시나리오

AD DS(Active Directory 도메인 Services) 환경에서 Azure Virtual Desktop 애플리케이션을 패키지하고 배포해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- MSIX 앱 패키지 준비 및 만들기
- Microsoft Entra DS 환경에서 Azure Virtual Desktop에 대한 MSIX 앱 연결 이미지 구현
- AD DS 환경에서 Azure Virtual Desktop에서 MSIX 앱 연결 구현

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## 지침

>**중요**: Microsoft는 **Azure AD **(** Azure Active Directory**)의 이름을 Microsoft Entra ID로 **변경했습니다**. 이 변경에 대한 자세한 내용은 Azure Active Directory의 [새 이름을 참조하세요](https://learn.microsoft.com/en-us/entra/fundamentals/new-name). 이는 지속적인 노력이므로 개별 연습을 단계별로 진행하면서 랩 명령과 인터페이스 요소 간에 불일치가 있는 인스턴스가 계속 발생할 수 있습니다. 이 사항을 고려합니다(특히 이 랩**에서 Microsoft Entra 커넥트** Azure Active Directory 커넥트** 새 이름을 **지정합니다).

### 연습 1: MSIX 앱 패키지 준비 및 만들기

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 세션 호스트 구성 준비
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. MSIX 패키징을 위해 Windows 10을 실행하는 Azure VM 준비
1. 서명 인증서 생성
1. 패키지에 소프트웨어 다운로드
1. MSIX 패키징 도구 설치
1. MSIX 패키지 만들기

#### 작업 1: Azure Virtual Desktop 세션 호스트 구성 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. 랩 컴퓨터와 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 시작되기까지 몇 분 정도 걸립니다. 

   >**참고**: 이전 랩의 첫 번째 작업(AVD 프로필 구현 및 관리)에서 az140-21-RG 리소스 그룹의 세션 호스트에서 PSRemoting을 사용하도록 설정한 경우 Azure VM이 시작될 때까지 기다리지 않고 다음 작업으로 직접 진행할 수 있습니다. az140-21-RG 리소스 그룹의 세션 호스트에서 이전에 PSRemoting을 사용하도록 설정하지 않은 경우 VM이 시작될 때까지 기다린 다음 다음 명령을 실행합니다.

1. Cloud Shell**의 **PowerShell 세션에서 다음을 실행하여 세션 호스트에서 PowerShell 원격을 사용하도록 설정합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### 작업 2: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

1. Azure Portal의 Azure Portal이 표시된 웹 브라우저 창 내의 Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택합니다. 그런 다음, **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json**파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 MSIX 패키지를 만들고 Microsoft Entra DS에 조인하는 데 사용할 Windows 10을 실행하는 Azure VM을 배포합니다기본

   ```powershell
   $vNetResourceGroupName = 'az140-11-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $vNetResourceGroupName).Location
   $resourceGroupName = 'az140-42-RG'
   New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0402vmDeployment `
     -TemplateFile $HOME/az140-42_azuredeploycl42.json `
     -TemplateParameterFile $HOME/az140-42_azuredeploycl42.parameters.json
   ```

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 10분 정도 걸릴 수 있습니다. 

#### 작업 3: MSIX 패키징을 위해 Windows 10을 실행하는 Azure VM 준비

1. 랩 컴퓨터의 Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 **머신** 블레이드의 가상 머신** 목록에서 az140-cl-vm42** 항목을 선택합니다**. 그러면 **az140-cl-vm42** 블레이드가 열립니다.
1. **az140-cl-vm42** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-cl-vm42 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 이 사용자 계정을 만들 때 설정한 사용자 이름 및 암호로 **wvdadmin1@adatum.com** 로그인합니다. 
1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 windows PowerShell ISE**를 관리자 권한으로 시작**합니다. 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 MSIX 패키징을 위한 운영 체제를 준비합니다.

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **참고**: 이러한 레지스트리의 마지막 변경 내용은 사용자 액세스 제어를 사용하지 않도록 설정합니다. 기술적으로는 필요하지 않지만 이 랩에 설명된 프로세스를 간소화합니다.

#### 작업 4: 서명 인증서 생성

> **참고**: 이 랩에서는 자체 서명된 인증서를 사용합니다. 프로덕션 환경에서는 의도한 용도에 따라 공용 인증 기관 또는 내부 인증 기관에서 발급한 인증서를 사용해야 합니다.

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 일반 이름 특성이 Adatum**으로 설정된 **자체 서명된 인증서를 생성하고 로컬 컴퓨터** 인증서 저장소의 **개인** 폴더에 **인증서를 저장**합니다.

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 로컬 컴퓨터 인증서 저장소를 대상으로 하는 인증서 콘솔을 시작**합니다**.

   ```powershell
   certlm.msc
   ```

1. 인증서 콘솔 창에서 개인** 폴더를 확장하고**, 인증서** 하위 폴더를 선택하고**, Adatum** 인증서를 마우스 오른쪽 단추로 클릭하고**, 마우스 오른쪽 단추로 클릭 메뉴에서 모든 작업**과 **내보내기를** 차례로 선택합니다**.** **  그러면 인증서 내보내기 마법사**가 **시작됩니다. 
1. **인증서 내보내기 마법사의 **인증서 내보내기 마법사**** 시작 페이지에서 다음**을 선택합니다**.
1. **인증서 내보내기 마법사**의 **프라이빗 키** 내보내기 페이지에서 예 옵션을 **선택하고 프라이빗 키** 옵션을 내보내고 다음**을 선택합니다**.
1. **인증서 내보내기 마법사**의 **파일 형식** 내보내기 페이지에서 검사box **모든 확장 속성** 내보내기, 검사 상자 **인증서 개인 정보** 사용, 다음** 선택 취소를 선택합니다**.
1. **인증서 내보내기 마법사**의 **보안** 페이지에서 암호** 검사 상자를 선택하고 **아래 텍스트 상자에 Pa55w.rd1234**를 입력**하고 다음**을 선택합니다**.
1. **인증서 내보내기 마법사**의 **내보낼 파일** 페이지 **파일 이름** 텍스트 상자에서 **찾아보기**를 선택합니다. 그런 후에 **다른 이름으로 저장** 대화 상자에서 **C:\\Allfiles\\Labs\\04** 폴더(필요한 경우 폴더 만들기)로 이동하여 **파일 이름** 텍스트 상자에 **adatum.pfx**를 입력하고 **저장**을 선택합니다.
1. **인증서 내보내기 마법사**의 **내보낼 파일** 페이지로 돌아와 텍스트 상자에 **C:\\Allfiles\\Labs\\04\\adatum.pfx** 항목이 포함되어 있는지 확인하고 **다음**을 선택합니다.
1. **인증서 내보내기 마법사의 **인증서 내보내기 완료 마법사**** 페이지에서 마침**을 선택하고 **확인을** 선택하여 **성공적인 내보내기를 승인합니다. 

   > **참고**: 자체 서명된 인증서를 사용하므로 대상 세션 호스트의 **신뢰할 수 있는 사람** 인증서 저장소에 설치해야 합니다.

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 대상 세션 호스트의 **신뢰할 수 있는 사람** 인증서 저장소에 새로 생성된 인증서를 설치합니다.

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   $cleartextPassword = 'Pa55w.rd1234'
   $securePassword = ConvertTo-SecureString $cleartextPassword -AsPlainText -Force
   $localPath = 'C:\Allfiles\Labs\04'
   ForEach ($wvdhost in $wvdhosts){
      $remotePath = "\\$wvdhost\C$\Allfiles\Labs\04\"
      New-Item -ItemType Directory -Path $remotePath -Force
      Copy-Item -Path "$localPath\adatum.pfx" -Destination $remotePath -Force
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Import-PFXCertificate -CertStoreLocation Cert:\LocalMachine\TrustedPeople -FilePath 'C:\Allfiles\Labs\04\adatum.pfx' -Password $using:securePassword
      } 
   }
   ```

#### 작업 5: 패키지에 소프트웨어 다운로드

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 Microsoft Edge**를 시작하고 .**https://github.com/microsoft/XmlNotepad******
1. **microsoft/XmlNotepad** **readme.md** 페이지에서 독립 실행형 다운로드 가능 설치 관리자 다운로드 링크를 선택하여 압축된 설치 파일을 다운로드합니다.
1. az140-cl-vm42에 대한 **Bastion 세션 내에서 파일 탐색기 시작하고, 다운로드** 폴더로 **이동하고, 압축된 파일을 열고, 압축된 파일의 폴더 내에서 콘텐츠를 복사한 다음, C:\\AllFiles\\Labs\\04**\\ 디렉터리에 붙여**넣**습니다. 

#### 작업 6: MSIX 패키징 도구 설치

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 Microsoft Store** 앱을 시작**합니다.
1. **Microsoft Store** 앱에서 MSIX 패키징 도구를** 검색하여 선택하고 **MSIX **패키징 도구** 페이지에서 가져오기**를 선택합니다**.
1. 메시지가 표시되면 로그인을 건너뛰고 설치가 완료될 때까지 기다립니다. 설치가 완료되면 **열기**를 선택하고 **진단 데이터 보내기** 대화 상자에서 **거절**을 선택합니다. 

#### 작업 7: MSIX 패키지 만들기

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 창으로 전환**하고 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Windows Search 서비스를 사용하지 않도록** 설정합니다.

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 MSIX 패키지를 호스트할 폴더를 만듭니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 추출된 설치 관리자 파일에서 Zone.Identifier 대체 데이터 스트림을 제거합니다. 이 파일의 값은 "3"이며, 이는 인터넷에서 다운로드되었음을 나타냅니다.

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 MSIX 패키징 도구** 인터페이스로 전환하고**, **작업** 선택 페이지에서 애플리케이션 패키지 - 앱 패키지** 항목 만들기를 선택합니다**.** 그러면 새 패키지** 만들기 마법사가 **시작됩니다.
1. **새 패키지** 만들기 마법사의 환경** 선택 페이지에서 이 컴퓨터**의 **패키지 만들기 옵션이 선택되어 있는지 확인하고 **다음**을 선택하고 **MSIX 패키징 도구 드라이버** 설치**를 기다립니다.
1. **새 패키지** 만들기 마법사의 **컴퓨터** 준비 페이지에서 권장 사항을 검토합니다. 보류 중인 재부팅이 있는 경우 운영 체제를 다시 시작하고, 계정을 사용하여 **wvdadmin1@adatum.com** 다시 로그인한 다음, 계속하기 전에 MSIX 패키징 도구를** 다시 시작**합니다. 

   >**참고**: MSIX 패키징 도구는 일시적으로 Windows 업데이트 및 Windows Search를 사용하지 않도록 설정합니다. 이 경우 Windows Search 서비스가 이미 비활성화되어 있습니다. 

1. 새 패키지** 만들기 마법사의 **컴퓨터** 준비 페이지에서 다음**을 클릭합니다**.**
1. **새 패키지 만들기** 마법사의 **설치 프로그램 선택** 페이지에서 **패키지하려는 설치 프로그램 선택** 텍스트 상자 옆에 있는 **찾아보기**를 선택합니다. 그런 다음, **열기** 대화 상자에서 **C:\\AllFiles\\Labs\\04** 폴더로 이동하여 **XmlNotepadSetup.msi**를 선택하고 **열기**를 클릭합니다. 
1. **새 패키지 만들기** 마법사의 **설치 프로그램 선택** 페이지에 있는 **서명 기본 설정** 드롭다운 목록에서 **인증서(.pfx)로 서명** 항목을 선택합니다. 그런 다음, **인증서 찾아보기** 텍스트 상자 옆에 있는 **찾아보기**를 선택하고 **열기** 대화 상자에서 **C:\\AllFiles\\Labs\\04** 폴더로 이동해 **adatum.pfx** 파일을 선택한 후 **열기**를 클릭합니다. **암호** 텍스트 상자에는 **Pa55w.rd1234**를 입력하고 **다음**을 클릭합니다.
1. **새 패키지 만들기 마법사의 **패키지** 정보** 페이지에서 패키지 정보를 검토하고 게시자 이름이 CN=Adatum**으로 설정되어 있는지 확인하고 다음**을 **선택합니다**. 그러면 다운로드한 소프트웨어의 설치가 트리거됩니다.
1. **XML메모장 설치** 창에서 사용권 계약 조건에 동의하고 설치**를 선택하고 **설치가 완료되면 XML 시작 메모장 검사** 상자를 선택하고 **마침**을 선택합니다**.
1. 메시지가 표시되면 XML 메모장 분석 창에서 **아니요**를 선택하고**, XML 메모장 실행 중인지 확인하고, 닫고, MSIX 패키징 도구** 창에서 **새 패키지** 만들기 마법사로 다시 **전환하고, 다음**을 선택합니다**.**

   > **참고**: 이 경우 설치를 완료하기 위해 다시 시작할 필요가 없습니다.

1. **새 패키지** 만들기 마법사의 **첫 번째 시작 작업** 페이지에서 제공된 정보를 검토하고 다음**을 선택합니다**.
1. 메시지가 표시 **되면 [예**]를 선택하고 **계속 진행합니다**.
1. **새 패키지** 만들기 마법사의 **서비스 보고서** 페이지에서 나열된 서비스가 없는지 확인하고 다음**을 선택합니다**.
1. **새 패키지 만들기** 마법사의 **패키지 만들기** 페이지 **위치 저장** 텍스트 상자에 **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix**를 입력하고 **만들기**를 클릭합니다.
1. 패키지가 **성공적으로 생성됨** 대화 상자에서 저장된 패키지의 위치를 적어 두고 닫기를** 선택합니다**.
1. 파일 탐색기 창으로 전환하여 **C:\\Allfiles\\Labs\\04\\XmlNotepad** 폴더로 이동한 후 *.msix 및 *.xml 파일이 있는지 확인합니다.
1. **XmlNotepad.msix** 파일을 **C:\\Allfiles\\Labs\\04** 폴더로 복사합니다.


### 연습 2: Microsoft Entra DS 환경에서 Azure Virtual Desktop에 대한 MSIX 앱 연결 이미지 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Window 10 Enterprise Edition을 실행하는 Azure VM에서 Hyper-V 사용
1. MSIX 앱 연결 이미지 만들기

#### 작업 1: Window 10 Enterprise Edition을 실행하는 Azure VM에서 Hyper-V 사용

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 MSIX 앱 연결을 위한 대상 Azure Virtual Desktop 호스트를 준비합니다. 

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
         reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
         reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
         reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
         reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
         Set-Service -Name wuauserv -StartupType Disabled
      }
   }
   ```

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Azure Virtual Desktop 호스트에 Hyper-V PowerShell 모듈을 포함하여 Hyper-V 및 해당 관리 도구를 설치합니다.

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. 대상 운영 체제를 다시 시작하라는 메시지가 표시되면 **예**를 선택합니다.
1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 로컬 컴퓨터에 Hyper-V PowerShell 모듈을 포함하여 Hyper-V 및 해당 관리 도구를 설치합니다.

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V 구성요소 설치가 완료되면 **예**를 선택하여 운영 체제를 다시 시작합니다. 다시 시작한 후 이 사용자 계정을 만들 때 설정한 사용자 이름 및 암호를 사용하여 다시 **wvdadmin1@adatum.com** 로그인합니다.

#### 작업 2: MSIX 앱 연결 이미지 만들기

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 Microsoft Edge**를 시작하고 **을 찾**https://aka.ms/msixmgr**습니다.** 그러면 **msixmgr.zip** 파일(MSIX 관리 도구 보관)이 **다운로드** 폴더에 자동으로 다운로드됩니다.
1. 파일 탐색기에서 **다운로드** 폴더로 이동하여 압축된 파일을 열고 **x64** 폴더의 내용을 **C:\\AllFiles\\Labs\\04** 폴더에 복사합니다. 
1. az140-cl-vm42에 대한 **Bastion 세션 내에서 관리자 권한으로 Windows PowerShell ISE**를 시작하고 **관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 MSIX 앱 연결 이미지로 사용할 VHD 파일을 만듭니**다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   New-VHD -SizeBytes 128MB -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Dynamic -Confirm:$false
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 새로 만든 VHD 파일을 탑재합니다.

   ```powershell
   $vhdObject = Mount-VHD -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Passthru
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 디스크를 초기화하고, 새 파티션을 만들고, 포맷하고, 사용 가능한 첫 번째 드라이브 문자를 할당합니다.

   ```powershell
   $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
   $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
   Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
   ```

   > **참고**: F: 드라이브의 서식을 지정하라는 팝업 창이 표시되면 취소**를 선택합니다**.

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 MSIX 파일을 호스트하고 이전 작업에서 만든 MSIX 패키지로 압축을 풀 폴더 구조를 만듭니다.

   ```powershell
   $appName = 'XmlNotepad'
   New-Item -ItemType Directory -Path "$($partition.DriveLetter):\Apps" -Force
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination "$($partition.DriveLetter):\Apps" -applyacls
   ```

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 파일 탐색기 F:\\Apps** 폴더로 **이동하여 해당 콘텐츠를 검토합니다. 폴더에 대한 액세스 권한을 얻으라는 메시지가 표시되면 **계속**을 선택합니다.
1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 MSIX 이미지로 사용할 VHD 파일을 분리합니다.

   ```powershell
   Dismount-VHD -Path "C:\Allfiles\Labs\04\MSIXVhds\$appName.vhd" -Confirm:$false
   ```

### 연습 3: Azure Virtual Desktop 세션 호스트에서 MSIX 앱 연결 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 호스트를 포함하는 Active Directory 그룹 구성
1. MSIX 앱 연결에 대한 Azure Files 공유 설정
1. Azure Virtual Desktop 세션 호스트에서 MSIX 앱 연결 이미지 탑재 및 등록
1. MSIX 앱을 애플리케이션 그룹에 게시
1. MSIX 앱 연결의 기능 유효성 검사

#### 작업 1: Azure Virtual Desktop 호스트를 포함하는 Active Directory 그룹 구성

1. 랩 컴퓨터로 전환하고, Azure Portal을 표시하는 웹 브라우저에서 가상 머신을 검색하여 선택하고 **, 가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작**** 합니다.
1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩에서 사용되는 Microsoft Entra 테넌트에 동기화될 AD DS 그룹 개체를 만듭니다.

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **참고**: 이 그룹을 사용하여 az140-42-msixvhds 파일 공유에 **Azure Virtual Desktop 호스트 권한을 부여합니다** .

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이전 단계에서 만든 그룹에 구성원을 추가합니다.

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 'az140-hosts-42-p1' 그룹의 멤버인 서버를 다시 시작합니다.

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **참고**: 이 단계는 그룹 멤버 자격 변경이 적용되도록 합니다. 

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 시작** 메뉴에서 Microsoft Entra 커넥트** 폴더를 확장하고 **Microsoft Entra 커넥트** 선택합니다**.****
1. **Microsoft Entra 커넥트** 창의 **Microsoft Entra 커넥트** 시작 페이지에서 구성**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **추가 작업** 페이지에서 동기화 옵션** 사용자 지정을 선택하고 **다음**을 선택합니다**.
1. **Microsoft Entra 커넥트 창의 Microsoft Entra**에 커넥트** 페이지에서 이 사용자 계정을 만들 때 설정한 암호로 이 작업의 앞부분에서 **식별한 aadsyncuser** 사용자 계정의 **사용자 계정 이름을 사용하여 인증합니다.
1. **Microsoft Entra 커넥트 창의 **디렉터리** 커넥트** 페이지에서 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **Do기본 및 OU 필터링** 페이지에서 선택한 동기화기본 및 OU** 옵션이 **선택되어 있는지 확인하고, adatum.com** 노드를 확장하고**, WVDInfra** OU 옆에 **있는 검사 상자를 선택하고(선택한 다른 검사 상자를 변경하지 않음) 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **선택적 기능** 페이지에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **Microsoft Entra 커넥트** 창의 **구성** 준비 페이지에서 구성이 완료되면** 검사box **동기화 프로세스를 시작하고 구성**을 선택합니다**.
1. 구성 완료** 페이지에서 정보를 검토하고 종료**를 **선택하여 **Microsoft Entra 커넥트** 창을 닫습니다**.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할과 함께 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. az140-dc-vm11**에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Active Directory**를 검색하고 선택하여 **이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Azure Active Directory 블레이드의 왼쪽**에 있는 세로 메뉴 모음의 관리** 섹션에서 그룹을** 클릭합니다**. 
1. **그룹 | 모든 그룹** 블레이드의 그룹 목록에서 **az140-hosts-42-p1** 항목을 선택합니다.

   > **참고**: 표시할 그룹에 대한 페이지를 새로 고쳐야 할 수 있습니다.

1. **az140-hosts-42-p1** 블레이드의 왼쪽에 있는 세로 **메뉴 모음의 관리** 섹션에서 구성원**을 클릭합니다**.
1. **az140-hosts-42-p1 | 구성원** 블레이드에서 **직접 구성원** 목록에 이 작업 앞부분에서 그룹에 추가한 Azure Virtual Desktop 풀의 호스트 3개가 포함되어 있는지 확인합니다.

#### 작업 2: MSIX 앱 연결에 대한 Azure Files 공유 설정

1. 랩 컴퓨터에서 Bastion 세션 **으로 다시 az140-cl-vm42**로 전환합니다.
1. az140-cl-vm42에 대한 **Bastion 세션 내에서 InPrivate 모드에서 Microsoft Edge를 시작하고, Azure Portal[로 이동](https://portal.azure.com)한 다음, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인**합니다.

   > **참고**: Microsoft Edge InPrivate 모드를 사용해야 합니다.

1. az140-cl-vm42**에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Storage 계정을** 검색하여 선택하고 **Storage 계정** 블레이드에서 **사용자 프로필을 호스트하도록 구성한 스토리지 계정을 선택합니다.

   > **참고**: 랩의 이 부분은 랩 Azure AD DS(Virtual Desktop 프로필 관리)** 또는 **Azure Virtual Desktop 프로필 관리(Microsoft Entra DS)를 완료**해야 합니다.**

   > **참고**: 프로덕션 시나리오에서는 별도의 스토리지 계정을 사용하는 것이 좋습니다. 이렇게 하려면 사용자 프로필을 호스팅하는 스토리지 계정에 대해 이미 구현한 Microsoft Entra DS 인증을 위해 해당 스토리지 계정을 구성해야 합니다. 동일한 스토리지 계정을 사용하여 개별 랩에서 중복 단계를 최소화합니다.

1. 스토리지 계정 블레이드의 왼쪽 세로 메뉴에서 IAM(Access Control)**을 선택합니다**.
1. 스토리지 계정의 **IAM(액세스 제어)** 블레이드에서 + 추가**를 선택하고 **드롭다운 메뉴에서 역할 할당** 추가를 선택합니다**. 
1. 역할 할당** 추가 블레이드에서 **다음 설정을 지정하고 저장**을 선택합니다**.

   |설정|값|
   |---|---|
   |역할|**스토리지 파일 데이터 SMB 공유 관리자 권한 기여자**|
   |다음에 대한 액세스 할당|**사용자, 그룹 또는 서비스 주체**|
   |선택|**az140-wvd-admins**|

   > **참고**: az140-wvd-admins** 그룹에는 **공유 권한을 구성하는 데 사용할 wvdadmin1** 사용자 계정이 포함되어 **있습니다. 

1. 이전 두 단계를 반복하여 다음 역할 할당을 구성합니다.

   |설정|값|
   |---|---|
   |역할|**스토리지 파일 데이터 SMB 공유 관리자 권한 기여자**|
   |다음에 대한 액세스 할당|**사용자, 그룹 또는 서비스 주체**|
   |선택|**az140-hosts-42-p1**|

   |설정|값|
   |---|---|
   |역할|**스토리지 파일 데이터 SMB 공유 읽기 권한자**|
   |다음에 대한 액세스 할당|**사용자, 그룹 또는 서비스 주체**|
   |선택|**az140-wvd-users**|

   > **참고**: Azure Virtual Desktop 사용자 및 호스트는 적어도 파일 공유에 대한 읽기 권한이 필요합니다.

1. 스토리지 계정 블레이드의 왼쪽에 있는 세로 **메뉴의 데이터 스토리지** 섹션에서 파일 공유**를 선택한 **다음+ 파일 공유**를 선택합니다**.
1. 새 파일 공유 블레이드에서 **다음 설정을 지정하고 만들기**를 선택합니다**(다른 설정은 기본값으로 유지**).

   |설정|값|
   |---|---|
   |속성|**az140-42-msixvhds**|

1. Azure Portal을 표시하는 Microsoft Edge의 파일 공유 목록에서 새로 만든 파일 공유를 선택합니다. 

1. Bastion 세션에서 **az140-cl-vm42**** 로 이동하고 명령 프롬프트**** 창에서 **다음을 실행하여 **az140-42-msixvhds** 공유에 드라이브를 매핑하고(자리 표시자를 스토리지 계정 이름으로 바꾸기`<storage-account-name>`) 명령이 성공적으로 완료되었는지 확인합니다.

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. Az140-cl-vm42**에 대한 **Bastion 세션 내에서 명령 프롬프트** 창에서 **다음을 실행하여 세션 호스트의 컴퓨터 계정에 필요한 NTFS 권한을 부여합니다.

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**참고**: wvdadmin1\@adatum.com 로그인**하는 동안 파일 탐색기 사용하여 이러한 권한을 설정할 수도 있습니다**. 

   >**참고**: 다음으로 MSIX 앱 연결 기능의 유효성을 검사합니다.

1. az140-cl-vm42**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 창에서 **다음을 실행하여 이전 연습에서 만든 VHD 파일을 이 연습의 앞부분에서 만든 Azure Files 공유에 복사합니다.

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages' -Force
   ```

#### 작업 3: Azure Virtual Desktop 세션 호스트에 MSIX 앱 연결 이미지 탑재 및 등록

1. az140-cl-vm42에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Virtual Desktop을 검색하여 선택하고 **, **Azure Virtual Desktop**** 블레이드의 왼쪽에 있는 세로 메뉴의 **관리** 섹션에서 호스트 풀을** 선택합니다**.**
1. **Azure Virtual Desktop \| 호스트 풀** 블레이드의 호스트 풀 목록에서 **az140-21-hp1** 항목을 선택합니다.
1. **az140-21-hp1 \| 속성** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **MSIX 패키지**를 선택합니다.
1. **az140-21-hp1 \| MSIX 패키지** 블레이드에서 **+ 추가**를 클릭합니다.
1. **MSIX 패키지 추가** 블레이드의 **MSIX 이미지 경로** 텍스트 상자에 **XmlNotepad.vhd** 파일에 대한 경로를 `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` 형식으로 입력한 다음(`<storage-account-name>` 자리 표시자를 **az140-42-msixvhds** 파일 공유를 호스트하는 스토리지 계정 이름으로 바꿈), **추가**를 클릭합니다.
1. MSIX 패키지** 추가 블레이드에서 **다음 설정을 지정하고 추가**를 클릭합니다**.

   |설정|값|
   |---|---|
   |MSIX 이미지 경로|**\\\\\<storage-account-name\>.file.core.windows.net az140-42-msixvhds Xml메모장.vhd**입니다. 여기서 자리 표시자는 `<storage-account-name>` az140-42-msixvhds 파일 공유를 호스팅하는 **스토리지 계정의 이름을 지정합니다**.\\\\|
   |MSIX 패키지|패키지를 만드는 동안 생성된 이름|
   |표시 이름|**XML 메모장**|
   |등록 유형|**주문형**|
   |State(상태)|**진행 중**|

#### 작업 4: 애플리케이션 그룹에 MSIX 앱 게시

> **참고**: MSIX 앱을 원격 앱과 데스크톱 앱 그룹 모두에 게시합니다. 

1. az140-cl-vm42에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Virtual Desktop을 검색하여 선택하고 **, **Azure Virtual Desktop**** 블레이드의 왼쪽에 있는 세로 메뉴의 **관리** 섹션에서 애플리케이션 그룹을** 선택합니다**.**
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 블레이드에서 **az140-21-hp1-Utilities-RAG** 애플리케이션 그룹 항목을 선택합니다.
1. **az140-21-hp1-Utilities-RAG** 블레이드의 왼쪽에 있는 세로 메뉴의 **관리** 섹션에서 애플리케이션을** 선택합니다**. 
1. **az140-21-hp1-Utilities-RAG \| 애플리케이션** 블레이드에서 + 추가**를 클릭합니다**.
1. 애플리케이션 추가 블레이드의 ****기본 및** **아이콘** 탭에서 다음 설정을 지정하고 저장**을 선택합니다**.**

   |설정|값|
   |---|---|
   |애플리케이션 소스|**앱 연결**|
   |Package(패키지)|이미지에 포함된 패키지를 나타내는 이름|
   |애플리케이션|**XMLNOTEPAD**|
   |애플리케이션 식별자|**XML 메모장**|
   |표시 이름|**XML 메모장**|
   |설명|**XML 메모장**|
   |아이콘 원본|**기본값**|

1. **Azure Virtual Desktop \| 애플리케이션 그룹** 블레이드로 다시 이동하여 **az140-21-hp1-DAG** 애플리케이션 그룹 항목을 선택합니다.
1. **az140-21-hp1-DAG** 블레이드의 왼쪽에 있는 세로 **메뉴의 관리** 섹션에서 애플리케이션을** 선택합니다**. 
1. **az140-21-hp1-DAG \| 애플리케이션** 블레이드에서 + 추가**를 클릭합니다**.
1. 애플리케이션** 추가 블레이드에서 **다음 설정을 지정하고 저장**을 선택합니다**.

   |설정|값|
   |---|---|
   |애플리케이션 소스|**MSIX 패키지**|
   |MSIX 패키지|이미지에 포함된 패키지를 나타내는 이름|
   |애플리케이션 이름|**XML 메모장**|
   |표시 이름|**XML 메모장**|
   |설명|**XML 메모장**|

#### 작업 5: MSIX 앱 연결의 기능 유효성 검사

1. az140-cl-vm42에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Windows Desktop 클라이언트 다운로드 페이지[로 ](https://go.microsoft.com/fwlink/?linkid=2068602)이동한 다음 다운로드가 완료되면 파일** 열기를 선택하여 **설치를 시작**합니다. **원격 데스크톱 설치** 마법사의 설치 범위** 페이지에서 이 컴퓨터**의 **모든 사용자에 대해 설치 옵션을 **선택하고 설치**를 클릭합니다**. 
1. 설치가 완료되면 설치가 **종료**될 때 원격 데스크톱 시작 검사 상자가 선택되어 있는지 확인하고 마침**을 클릭하여 **원격 데스크톱 클라이언트를 시작합니다.
1. **Remote Desktop** 클라이언트 창에서 **구독**을 선택하고 메시지가 표시되면 **aduser1** 사용자 계정 이름으로 로그인합니다. 암호로는 이 사용자 계정을 만들 때 설정한 암호를 사용합니다. 
1. 메시지가 표시되면 모든 앱**에 로그인된 상태로 유지 창에서 **조직에서 내 디바이스**를 관리할 수 있도록 허용 검사box를 선택 취소**하고 [아니요]를 클릭하고 **이 앱에만** 로그인합니다.
1. 원격 데스크톱 클라이언트 창의 **az140-21-ws1** 섹션 내에서 **XML 메모장** 아이콘을 두 번 클릭하고**, 메시지가 표시되면 암호를 제공하고, XML 메모장 성공적으로 시작되었는지 확인합니다.**


### 연습 4: 랩에서 프로비전되고 사용되는 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전되고 사용되는 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에서 프로비전되고 사용되는 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전되고 사용되는 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만들고 사용하는 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
