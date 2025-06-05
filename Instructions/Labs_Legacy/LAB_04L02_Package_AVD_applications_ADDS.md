---
lab:
  title: '랩: Azure Virtual Desktop 애플리케이션 패키징(AD DS)'
  module: 'Module 4: Manage User Environments and Apps'
---

# 랩 - Azure Virtual Desktop 애플리케이션 패키징(AD DS)
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩
- 완료된 랩 **AVD에 대한 조건부 액세스 정책 구성(AD DS)**
- 완료된 랩 **AVD 프로필 구현 및 관리(AD DS)**

## 예상 소요 시간

60분

## 랩 시나리오

Active Directory Domain Services(AD DS) 환경에서 Azure Virtual Desktop 애플리케이션 패키징과 배포를 수행해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- MSIX 앱 패키지 준비 및 만들기
- AD DS 환경에서 Azure Virtual Desktop용 MSIX 앱 연결 이미지 구현
- AD DS 환경의 Azure Virtual Desktop에서 MSIX 앱 연결 구현

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json
-  \\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json

## 지침

>**중요**: Microsoft는 **Azure Active Directory**(**Azure AD**)의 이름을 **Microsoft Entra ID**로 변경했습니다. 이 변경 내용에 대한 자세한 내용은 [Azure Active Directory의 새 이름](https://learn.microsoft.com/en-us/entra/fundamentals/new-name)을 참조하세요. 이는 지속적인 활동이므로 개별 연습을 진행하면서 랩 지침과 인터페이스 요소가 일치하지 않는 경우가 여전히 발생할 수 있습니다. 다음 사항을 고려합니다. 특히 이 랩에서는, **Microsoft Entra Connect**가 **Azure Active Directory Connect**의 새 이름을 지정합니다.

### 연습 1: MSIX 앱 패키지 준비 및 만들기

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 세션 호스트의 구성 준비
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. MSIX 패키징용으로 Windows 10을 실행하는 Azure VM 준비
1. 서명 인증서 생성
1. 패키징할 소프트웨어 다운로드
1. MSIX Packaging Tool 설치
1. MSIX 패키지 만들기

#### 작업 1: Azure Virtual Desktop 세션 호스트의 구성 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저 창에서 **Cloud Shell** 창 내에 **PowerShell** 셸 세션을 엽니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 사용할 Azure Virtual Desktop 세션 호스트 Azure VM을 시작합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Start-AzVM -NoWait
   ```

   > **참고**: 명령은 비동기적으로 실행되므로(-NoWait 매개 변수에 의해 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수는 있지만, Azure VM이 실제로 시작되려면 몇 분 정도 걸립니다. 

   > **참고**: 이전 랩의 첫 번째 작업(AVD 프로필 구현 및 관리)에서 az140-21-RG 리소스 그룹의 세션 호스트에 PSRemoting을 사용하도록 설정한 경우 Azure VM이 시작될 때까지 기다리지 않고 다음 작업을 직접 진행할 수 있습니다. 이전에 az140-21-RG 리소스 그룹의 세션 호스트에 PSRemoting을 사용하도록 설정하지 않은 경우 VM이 시작될 때까지 기다린 후 다음 명령을 실행합니다.

1. **Cloud Shell**의 PowerShell 세션에서 다음을 실행하여 세션 호스트에서 PowerShell Remoting을 사용하도록 설정합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Enable-AzVMPSRemoting
   ```
   
#### 작업 2: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

1. Azure Portal의 Azure Portal이 표시된 웹 브라우저 창 내의 Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택합니다. 그런 다음, **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\04\\az140-42_azuredeploycl42.parameters.json**파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 Windows 10을 실행하는 Azure VM을 배포합니다. 이 VM은 MSIX 패키지를 만든 후 Microsoft Entra DS 도메인에 조인하는 데 사용됩니다.

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

#### 작업 3: MSIX 패키징용으로 Windows 10을 실행하는 Azure VM 준비

1. 랩 컴퓨터에 표시된 Azure Portal에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드의 가상 머신 목록에서 **az140-cl-vm42** 항목을 선택합니다. 그러면 **az140-cl-vm42** 블레이드가 열립니다.
1. **az140-cl-vm42** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **Bastion을 통해 연결**을 선택합니다.
1. 메시지가 표시되면 이 사용자 계정을 만들 때 설정한 **wvdadmin1@adatum.com** 사용자 이름과 암호를 사용하여 로그인합니다. 
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 관리자로 **Windows PowerShell ISE**를 시작하고, **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 MSIX 패키징용 운영 체제를 준비합니다.

   ```powershell
   Schtasks /Change /Tn "\Microsoft\Windows\WindowsUpdate\Scheduled Start" /Disable
   reg add HKLM\Software\Policies\Microsoft\WindowsStore /v AutoDownload /t REG_DWORD /d 0 /f
   reg add HKCU\Software\Microsoft\Windows\CurrentVersion\ContentDeliveryManager /v PreInstalledAppsEnabled /t REG_DWORD /d 0 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ContentDeliveryManager\Debug /v ContentDeliveryAllowedOverride /t REG_DWORD /d 0x2 /f
   reg add HKLM\Software\Microsoft\RDInfraAgent\MSIXAppAttach /v PackageListCheckIntervalMinutes /t REG_DWORD /d 1 /f
   reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /t REG_DWORD /d 0 /f
   ```

   > **참고**: 위의 레지스트리 변경 명령 중 마지막 명령에서 사용자 액세스 제어가 사용하지 않도록 설정됩니다. 기술적으로는 반드시 이렇게 설정하지 않아도 되지만, 이렇게 설정하면 이 랩에서 설명하는 프로세스를 간편하게 진행할 수 있습니다.

#### 작업 4: 서명 인증서 생성

> **참고**: 이 랩에서는 자체 서명된 인증서를 사용합니다. 프로덕션 환경에서는 용도에 따라 공용 인증 기관이나 내부 인증 기관에서 발급한 인증서를 사용해야 합니다.

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 일반 이름 특성을 **Adatum**으로 설정한 자체 서명된 인증서를 생성한 후 **로컬 머신** 인증서 저장소의 **개인** 폴더에 저장합니다.

   ```powershell
   New-SelfSignedCertificate -Type Custom -Subject "CN=Adatum" -KeyUsage DigitalSignature -KeyAlgorithm RSA -KeyLength 2048 -CertStoreLocation "cert:\LocalMachine\My"
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 로컬 머신 인증서 저장소를 대상으로 **인증서** 콘솔을 시작합니다.

   ```powershell
   certlm.msc
   ```

1. **Certificates** 콘솔 창에서 **Personal** 폴더를 확장하고 **Certificates** 하위 폴더를 선택합니다. 그런 다음 **Adatum** 인증서를 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **모든 작업**, **내보내기**를 차례로 선택합니다. 그러면 **인증서 내보내기 마법사**가 시작됩니다. 
1. **인증서 내보내기 마법사**의 **인증서 내보내기 마법사 시작** 페이지에서 **다음**을 선택합니다.
1. **인증서 내보내기 마법사**의 **프라이빗 키 내보내기** 페이지에서 **예, 프라이빗 키를 내보냅니다.** 옵션을 선택하고 **다음**을 선택합니다.
1. **인증서 내보내기 마법사**의 **내보내기 파일 형식** 페이지에서 **확장 속성 모두 내보내기** 체크박스를 선택하고 **인증서 개인 정보 사용** 체크박스의 선택을 취소한 후에 **다음**을 선택합니다.
1. **인증서 내보내기 마법사**의 **보안** 페이지에서 **암호** 체크박스를 선택하고 아래쪽 텍스트 상자에 **Pa55w.rd1234**를 입력한 후에 **다음**을 선택합니다.
1. **인증서 내보내기 마법사**의 **내보낼 파일** 페이지 **파일 이름** 텍스트 상자에서 **찾아보기**를 선택합니다. 그런 후에 **다른 이름으로 저장** 대화 상자에서 **C:\\Allfiles\\Labs\\04** 폴더(필요한 경우 폴더 만들기)로 이동하여 **파일 이름** 텍스트 상자에 **adatum.pfx**를 입력하고 **저장**을 선택합니다.
1. **인증서 내보내기 마법사**의 **내보낼 파일** 페이지로 돌아와 텍스트 상자에 **C:\\Allfiles\\Labs\\04\\adatum.pfx** 항목이 포함되어 있는지 확인하고 **다음**을 선택합니다.
1. **인증서 내보내기 마법사**의 **인증서 내보내기 마법사 완료** 페이지에서 **마침**, **확인**을 차례로 선택하여 파일 내보내기가 정상적으로 완료되었음을 승인합니다. 

   > **참고**: 여기서는 자체 서명된 인증서를 사용하므로 대상 세션 호스트의 **신뢰할 수 있는 사용자** 인증서 저장소에 해당 인증서를 설치해야 합니다.

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 대상 세션 호스트의 **신뢰할 수 있는 사용자** 인증서 저장소에 새로 생성한 인증서를 설치합니다.

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

#### 작업 5: 패키징할 소프트웨어 다운로드

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **Microsoft Edge**를 시작하고 **https://github.com/microsoft/XmlNotepad**로 이동합니다.
1. **microsoft/XmlNotepad** **readme.md** 페이지에서 독립 실행형 다운로드 가능 설치 관리자 다운로드 링크를 선택하여 압축된 설치 파일을 다운로드합니다.
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 파일 탐색기를 시작하고 **다운로드** 폴더로 이동합니다. 그런 다음, 압축된 파일을 열어 압축 파일의 폴더 내에서 콘텐츠를 복사하여 **C:\\AllFiles\\Labs\\04\\** 디렉터리에 붙여넣습니다. 

#### 작업 6: MSIX Packaging Tool 설치

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **Microsoft Store** 앱을 시작합니다.
1. **Microsoft Store** 앱에서 **MSIX Packaging Tool**을 검색하여 선택하고 **MSIX Packaging Tool** 페이지에서 **받기**를 선택합니다.
1. 메시지가 표시되면 로그인을 건너뛰고 설치가 완료될 때까지 기다립니다. 설치가 완료되면 **시작**을 선택하고 **진단 데이터 보내기** 대화 상자에서 **거절**을 선택합니다. 

#### 작업 7: MSIX 패키지 만들기

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 창으로 전환한 다음, **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 Windows Search 서비스를 사용하지 않도록 설정합니다.

   ```powershell
   $serviceName = 'wsearch'
   Set-Service -Name $serviceName -StartupType Disabled
   Stop-Service -Name $serviceName
   ```

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 MSIX 패키지를 호스트할 폴더를 만듭니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\AllFiles\Labs\04\XmlNotepad' -Force
   ```

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 압축을 푼 설치 관리자 파일에서 Zone.Identifier 대체 데이터 스트림을 제거합니다. 이 데이터 스트림의 값은 해당 스트림을 인터넷에서 다운로드했음을 나타내는 3입니다.

   ```powershell
   Get-ChildItem -Path 'C:\AllFiles\Labs\04' -Recurse -File | Unblock-File
   ```

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **MSIX 패키지 도구** 인터페이스로 전환하고 **작업 선택** 페이지에서 **애플리케이션 패키지 - 앱 패키지 만들기** 항목을 선택합니다. 그러면 **새 패키지 만들기** 마법사가 시작됩니다.
1. **새 패키지 만들기** 마법사의 **환경 선택** 페이지에서 **이 컴퓨터에 패키지 만들기** 옵션이 선택되어 있는지 확인하고 **다음**을 선택한 후 **MSIX 패키지 도구 드라이버**가 설치될 때까지 기다립니다.

   > **참고**: MSIX 패키지 도구 드라이버를 설치하는 데 5~10분이 소요됩니다. 상태 열에 처음에는 **확인 중**으로 표시되고, 설치되면 **설치됨**으로 표시됩니다.

1. **새 패키지 만들기** 마법사의 **컴퓨터 준비** 페이지에서 추천 항목을 검토합니다. 다시 부팅이 보류 중인 경우 운영 체제를 다시 시작하고 계속하기 전에 **wvdadmin1@adatum.com** 계정을 사용하여 다시 로그인한 다음 **MSIX 패키지 도구**를 다시 시작합니다. 

   >**참고**: MSIX Packaging Tool 사용 시에는 Windows 업데이트와 Windows Search가 일시적으로 사용하지 않도록 설정됩니다. 여기서는 Windows Search 서비스를 이미 사용하지 않도록 설정했습니다. 

1. **새 패키지 만들기** 마법사의 **컴퓨터 준비** 페이지에서 **다음**을 클릭합니다.
1. **새 패키지 만들기** 마법사의 **설치 프로그램 선택** 페이지에서 **패키지하려는 설치 프로그램 선택** 텍스트 상자 옆에 있는 **찾아보기**를 선택합니다. 그런 다음, **열기** 대화 상자에서 **C:\\AllFiles\\Labs\\04** 폴더로 이동하여 **XmlNotepadSetup.msi**를 선택하고 **열기**를 클릭합니다. 
1. **새 패키지 만들기** 마법사의 **설치 프로그램 선택** 페이지에 있는 **서명 기본 설정** 드롭다운 목록에서 **인증서(.pfx)로 서명** 항목을 선택합니다. 그런 다음, **인증서 찾아보기** 텍스트 상자 옆에 있는 **찾아보기**를 선택하고 **열기** 대화 상자에서 **C:\\AllFiles\\Labs\\04** 폴더로 이동해 **adatum.pfx** 파일을 선택한 후 **열기**를 클릭합니다. **암호** 텍스트 상자에는 **Pa55w.rd1234**를 입력하고 **다음**을 클릭합니다.
1. **새 패키지 만들기** 마법사의 **패키지 정보** 페이지에서 패키지 정보를 검토하고 게시자 이름이 **CN=Adatum**으로 설정되어 있는지 확인한 후에 **다음**을 선택합니다.
1. **패키지에 적용할 가속기 선택** 페이지에서 **다음**을 선택합니다. 그러면 다운로드한 소프트웨어의 설치가 트리거됩니다.
1. **XMLNotepad 설정** 창에서 라이선스 계약의 사용 약관에 동의하고 **설치**를 선택한 후 설치가 완료되면 **마침**을 선택합니다.
1. **새 패키지 만들기** 마법사의 **설치** 페이지에서 **다음**을 선택합니다.
1. **새 패키지 만들기** 마법사의 **첫 번째 시작 작업 관리** 페이지에서 제공된 정보를 검토하고 **다음**을 선택합니다.
1. **Are you done?** 메시지가 표시되면 **Yes, move on**을 선택합니다.
1. **새 패키지 만들기** 마법사의 **서비스 보고서** 페이지에서 아무 서비스도 표시되지 않음을 확인하고 **다음**을 선택합니다.
1. **새 패키지 만들기** 마법사의 **패키지 만들기** 페이지 **위치 저장** 텍스트 상자에 **C:\\Allfiles\\Labs\\04\\XmlNotepad\XmlNotepad.msix**를 입력하고 **만들기**를 클릭합니다.
1. **패키지 만들기 성공** 대화 상자에서 저장된 패키지의 위치를 확인하고 **닫기**를 선택합니다.
1. 파일 탐색기 창으로 전환하여 **C:\\Allfiles\\Labs\\04\\XmlNotepad** 폴더로 이동한 후 *.msix 및 *.xml 파일이 있는지 확인합니다.
1. **XmlNotepad.msix** 파일을 **C:\\Allfiles\\Labs\\04** 폴더로 복사합니다.


### 연습 2: AD DS 환경에서 Azure Virtual Desktop용 MSIX 앱 연결 이미지 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Window 11 Enterprise 다중 세션을 실행하는 Azure VM에서 Hyper-V 사용
1. MSIX 앱 연결 이미지 만들기

#### 작업 1: Window 11 Enterprise 다중 세션을 실행하는 Azure VM에서 Hyper-V 사용

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 MSIX 앱 연결용 대상 Azure Virtual Desktop 호스트를 준비합니다. 

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

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Hyper-V 및 해당 관리 도구(Azure Virtual Desktop 호스트의 Hyper-V PowerShell 모듈 포함)를 설치합니다.

   ```powershell
   $wvdhosts = 'az140-21-p1-0','az140-21-p1-1','az140-21-p1-2'
   ForEach ($wvdhost in $wvdhosts){
      Invoke-Command -ComputerName $wvdhost -ScriptBlock {
         Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
      }
   }
   ```

1. 대상 운영 체제를 다시 시작하라는 메시지가 표시되면 **예**를 선택합니다.
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Hyper-V 및 해당 관리 도구(로컬 컴퓨터의 Hyper-V PowerShell 모듈 포함)를 설치합니다.

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
   ```

1. Hyper-V 구성요소 설치가 완료되면 **예**를 선택하여 운영 체제를 다시 시작합니다. 다시 시작한 후 이 사용자 계정을 만들 때 설정한 **wvdadmin1@adatum.com** 사용자 이름과 암호를 사용하여 다시 로그인합니다.

#### 작업 2: MSIX 앱 연결 이미지 만들기

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **Microsoft Edge**를 시작하고 **https://aka.ms/msixmgr**로 이동합니다. 그러면 **msixmgr.zip** 파일(MSIX 관리 도구 보관)이 **다운로드** 폴더에 자동으로 다운로드됩니다.
1. 파일 탐색기에서 **다운로드** 폴더로 이동하여 압축된 파일을 열고 **x64** 폴더의 내용을 **C:\\AllFiles\\Labs\\04** 폴더에 복사합니다. 
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 관리자로 **Windows PowerShell ISE**를 시작하고 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 MSIX 앱 연결 이미지를 저장할 폴더를 만듭니다.

   ```powershell
   New-Item -ItemType Directory -Path 'C:\Allfiles\Labs\04\MSIXVhds' -Force
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 MSIX 파일을 호스팅할 VHD를 만들고 이전 작업에서 만든 MSIX 패키지의 압축을 풉니다.

   ```powershell
   $appName = 'XmlNotepad'
   Set-Location -Path 'C:\AllFiles\Labs\04\x64'
   .\msixmgr.exe -Unpack -packagePath ..\$appName.msix -destination ..\MSIXVhds\$appName.vhd -applyacls -create -filetype vhd -vhdSize 128 -rootDirectory Apps
   ```

1. **az140-cl-vm42**에 연결된 Bastion 세션 내의 파일 탐색기에서 **C:\AllFiles\Labs\04\MSIXVhds** 폴더로 이동하여 XmlNotepad.vhd라는 가상 디스크가 있는지 확인합니다.

### 연습 3: Azure Virtual Desktop 세션 호스트에서 MSIX 앱 연결 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Virtual Desktop 호스트가 포함된 Active Directory 그룹 구성
1. MSIX 앱 연결용 Azure Files 공유 설정
1. Azure Virtual Desktop 세션 호스트에서 MSIX 앱 연결 이미지 탑재 및 등록
1. MSIX 앱을 애플리케이션 그룹에 게시
1. MSIX 앱 연결의 기능 유효성 검사

#### 작업 1: Azure Virtual Desktop 호스트가 포함된 Active Directory 그룹 구성

1. 랩 컴퓨터로 전환하여 Azure Portal이 표시된 웹 브라우저에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-dc-vm11**을 선택합니다.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **Bastion을 통해 연결**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. **az140-dc-vm11**에 연결된 Bastion 세션 내에서 관리자로 **Windows PowerShell ISE**를 시작합니다.
1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩에서 사용되는 Microsoft Entra 테넌트에 동기화될 AD DS 그룹 개체를 만듭니다.

   ```powershell
   $ouPath = "OU=WVDInfra,DC=adatum,DC=com"
   New-ADGroup -Name 'az140-hosts-42-p1' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

   > **참고**: 이 그룹을 사용하여 **az140-42-msixvhds** 파일 공유에 대한 권한을 Azure Virtual Desktop 호스트에 부여합니다.

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 이전 단계에서 만든 그룹에 구성원을 추가합니다.

   ```powershell
   Get-ADGroup -Identity 'az140-hosts-42-p1' | Add-AdGroupMember -Members 'az140-21-p1-0$','az140-21-p1-1$','az140-21-p1-2$'
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 ‘az140-hosts-42-p1’ 그룹 구성원인 서버를 다시 시작합니다.

   ```powershell
   $hosts = (Get-ADGroup -Identity 'az140-hosts-42-p1' | Get-ADGroupMember | Select-Object Name).Name
   $hosts | ForEach-Object {Restart-Computer -ComputerName $_ -Force}
   ```

   > **참고**: 이 단계를 수행하면 그룹 등록 변경 내용이 적용됩니다. 

1. **az140-dc-vm11**에 연결된 Bastion 세션 내의 **시작** 메뉴에서 **Microsoft Azure AD Connect** 폴더를 확장하고 **Microsoft Azure AD Connect**를 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **Azure AD Connect 시작** 페이지에서 **구성**을 선택합니다.
1. **Microsoft Entra Connect** 창의 **추가 작업** 페이지에서 **동기화 옵션 사용자 지정**을 선택하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **Microsoft Entra에 연결** 페이지에서 이 작업의 앞부분에서 확인한 **aadsyncuser** 사용자 계정의 사용자 계정 이름과 이 사용자 계정을 만들 때 설정한 암호를 사용하여 인증합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **디렉터리 연결** 페이지에서 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **도메인 및 OU 필터링** 페이지에서 **선택한 도메인 및 OU 동기화** 옵션이 선택되어 있는지 확인합니다. 그런 다음 **adatum.com** 노드를 확장하여 **WVDInfra** OU 옆의 체크박스를 선택하고(선택되어 있는 나머지 모든 체크박스는 변경하지 않고 그대로 유지) **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **선택적 기능** 페이지에서 기본 설정을 적용하고 **다음**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 창의 **구성 준비 완료** 페이지에서 **구성이 완료되면 동기화 프로세스를 시작합니다.** 체크박스가 선택되었는지 확인하고 **구성**을 선택합니다.
1. **구성 완료** 페이지의 정보를 검토하고 **끝내기**를 선택하여 **Microsoft Azure Active Directory Connect** 창을 닫습니다.
1. **az140-dc-vm11**에 연결된 Bastion 세션 내에서 Microsoft Edge를 시작하고 [Azure Portal](https://portal.azure.com)로 이동합니다. 메시지가 표시되면 이 랩에서 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. **az140-dc-vm11**에 연결된 Bastion 세션 내의 Azure Portal을 표시하는 Microsoft Edge 창에서 **Microsoft Entra ID**를 검색하고 선택하여 이 랩에 사용 중인 Azure 구독과 연결된 Microsoft Entra 테넌트로 이동합니다.
1. Microsoft Entra ID 블레이드의 왼쪽 세로 메뉴 모음에 있는 **관리** 섹션에서 **그룹**을 클릭합니다. 
1. **그룹 | 모든 그룹** 블레이드의 그룹 목록에서 **az140-hosts-42-p1** 항목을 선택합니다.

   > **참고**: 표시할 그룹의 페이지를 새로 고쳐야 할 수 있습니다.

1. **az140-hosts-42-p1** 블레이드 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **구성원**을 선택합니다.
1. **az140-hosts-42-p1 | 구성원** 블레이드에서 **직접 구성원** 목록에 이 작업 앞부분에서 그룹에 추가한 Azure Virtual Desktop 풀의 호스트 3개가 포함되어 있는지 확인합니다.

#### 작업 2: MSIX 앱 연결용 Azure Files 공유 설정

1. 랩 컴퓨터에서 Bastion 세션으로 다시 전환하여 **az140-cl-vm42**로 전환합니다.
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 InPrivate 모드로 Microsoft Edge를 시작하여 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 랩에서 사용할 구독의 Owner 역할이 할당된 사용자 계정의 자격 증명을 입력하여 로그인합니다.

   > **참고**: Microsoft Edge InPrivate 모드를 사용해야 합니다.

1. **az140-cl-vm42**에 연결된 Bastion 세션 내의 Azure Portal이 표시된 Microsoft Edge 창에서 **스토리지 계정**을 검색하여 선택합니다. 그런 다음 **스토리지 계정** 블레이드에서 사용자 프로필 호스트용으로 구성한 스토리지 계정을 선택합니다.

   > **참고**: 이 랩 부분에서는 **AVD용 스토리지 구현 및 관리(AD DS)** 또는 **AVD용 스토리지 구현 및 관리(Microsoft Entra DS)** 랩을 완료할 수 있습니다.

   > **참고**: 프로덕션 시나리오에서 별도의 스토리지 계정 사용을 고려해야 합니다. 이를 위해서는 사용자 프로필을 호스팅하는 스토리지 계정에 대해 이미 구현한 Microsoft Entra DS 인증을 위한 해당 스토리지 계정을 구성해야 합니다. 이 과정에서는 개별 랩의 중복 단계를 최소화하기 위해 같은 스토리지 계정을 사용합니다.

1. 스토리지 계정 블레이드 왼쪽의 세로 메뉴에서 **액세스 제어(IAM)** 를 선택합니다.
1. 스토리지 계정의 **액세스 제어(IAM)** 블레이드에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다. 
1. **역할 할당 추가** 블레이드의 **역할** 탭에서 다음 설정을 할당하고 **다음**을 선택합니다.

   |설정|값|
   |---|---|
   |작업 함수 역할|**Storage 파일 데이터 SMB 공유 기여자**|

1. **역할 할당 추가** 블레이드의 **멤버** 탭에서 **+ 멤버 선택**을 ​​클릭하고 다음 설정을 할당한 후 **선택**을 클릭합니다. 

   |설정|값|
   |---|---|
   |선택|**az140-wvd-users**|

1. **역할 할당 추가** 블레이드에서 **검토 + 할당**을 선택한 다음 **검토 + 할당**을 선택합니다.
1. 스토리지 계정의 **액세스 제어(IAM)** 블레이드에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다. 
1. **역할 할당 추가** 블레이드의 **역할** 탭에서 다음 설정을 할당하고 **다음**을 선택합니다.

   |설정|값|
   |---|---|
   |작업 함수 역할|**Storage 파일 데이터 SMB 공유 높은 권한 기여자**|

1. **역할 할당 추가** 블레이드의 **멤버** 탭에서 **+ 멤버 선택**을 ​​클릭하고 다음 설정을 할당한 후 **선택**을 클릭합니다. 

   |설정|값|
   |---|---|
   |선택|**az140-wvd-admins**|

1. **역할 할당 추가** 블레이드에서 **검토 + 할당**을 선택한 다음 **검토 + 할당**을 선택합니다.
1. 스토리지 계정의 **액세스 제어(IAM)** 블레이드에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다. 
1. **역할 할당 추가** 블레이드의 **역할** 탭에서 다음 설정을 할당하고 **다음**을 선택합니다.

   |설정|값|
   |---|---|
   |작업 함수 역할|**Storage 파일 데이터 SMB 공유 높은 권한 기여자**|

1. **역할 할당 추가** 블레이드의 **멤버** 탭에서 **+ 멤버 선택**을 ​​클릭하고 다음 설정을 할당한 후 **선택**을 클릭합니다. 

   |설정|값|
   |---|---|
   |선택|**az140-hosts-42-p1**|

1. **역할 할당 추가** 블레이드에서 **검토 + 할당**을 선택한 다음 **검토 + 할당**을 선택합니다.

1. 스토리지 계정 블레이드 왼쪽의 세로 메뉴에 있는 **데이터 스토리지** 섹션에서 **파일 공유**, **+ 파일 공유**를 차례로 선택합니다.
1. **새 파일 공유** 블레이드에서 다음 설정을 지정하고 **다음: 백업 >** 을 선택합니다(다른 설정은 기본값 유지).

   |설정|값|
   |---|---|
   |속성|**az140-42-msixvhds**|
   |액세스 계층|**트랜잭션 최적화됨**|

1. **백업** 블레이드에서, **백업 사용** 확인란의 선택을 취소하고, **검토 + 만들기**를 선택하고, 유효성 검사 프로세스가 완료될 때까지 기다린 다음 **만들기**를 선택합니다.
1. Azure Portal이 표시된 Microsoft Edge의 파일 공유 목록에서 새로 만든 파일 공유를 선택합니다. 
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **명령 프롬프트**를 시작하고 **명령 프롬프트** 창에서 다음 명령을 실행하여 e **az140-42-msixvhds** 공유에 드라이브를 매핑합니다(`<storage-account-name>` 자리 표시자는 스토리지 계정 이름으로 바꿔야 함). 그런 다음, 명령이 정상적으로 완료되는지 확인합니다.

   ```cmd
   net use Z: \\<storage-account-name>.file.core.windows.net\az140-42-msixvhds
   ```

1. **az140-cl-vm42**에 연결된 Bastion 세션 내의 **명령 프롬프트** 창에서 다음을 실행하여 세션 호스트의 컴퓨터 계정에 필요한 NTFS 권한을 부여합니다.

   ```cmd
   icacls Z:\ /grant ADATUM\az140-hosts-42-p1:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-users:(OI)(CI)(RX) /T
   icacls Z:\ /grant ADATUM\az140-wvd-admins:(OI)(CI)(F) /T
   ```

   >**참고**: **wvdadmin1\@adatum.com**으로 로그인한 상태에서 파일 탐색기를 사용하여 이러한 권한을 설정할 수도 있습니다. 

   >**참고**: 다음으로는 MSIX 앱 연결의 기능 유효성을 검사합니다.

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **관리자: Windows PowerShell ISE** 창에서 다음 명령을 실행하여 이 연습의 앞부분에서 만든 Azure Files 공유에 이전 연습에서 만든 VHD 파일을 복사합니다.

   ```powershell
   New-Item -ItemType Directory -Path 'Z:\packages' 
   Copy-Item -Path 'C:\Allfiles\Labs\04\MSIXVhds\XmlNotepad.vhd' -Destination 'Z:\packages\' -Force
   ```

#### 작업 3: Azure Virtual Desktop 세션 호스트에서 MSIX 앱 연결 이미지 탑재 및 등록

1. **az140-cl-vm42**에 연결된 Bastion 내의 Azure Portal이 표시된 Microsoft Edge 창에서 **Azure Virtual Desktop**을 검색하여 선택합니다. 그런 다음 **Azure Virtual Desktop** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **호스트 풀**을 선택합니다.
1. **Azure Virtual Desktop \| 호스트 풀** 블레이드의 호스트 풀 목록에서 **az140-21-hp1** 항목을 선택합니다.
1. **az140-21-hp1 \| 속성** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **MSIX 패키지**를 선택합니다.
1. **az140-21-hp1 \| MSIX 패키지** 블레이드에서 **+ 추가**를 클릭합니다.
1. **MSIX 패키지 추가** 블레이드의 **MSIX 이미지 경로** 텍스트 상자에 **XmlNotepad.vhd** 파일에 대한 경로를 `\\<storage-account-name>.file.core.windows.net\az140-42-msixvhds\packages\XmlNotepad.vhd` 형식으로 입력한 다음(`<storage-account-name>` 자리 표시자를 **az140-42-msixvhds** 파일 공유를 호스트하는 스토리지 계정 이름으로 바꿈), **추가**를 클릭합니다.
1. **MSIX 패키지 추가** 블레이드에서 다음 설정을 지정하고 **추가**를 클릭합니다.

   |설정|값|
   |---|---|
   |MSIX 이미지 경로|**\\\\\<storage-account-name\>.file.core.windows.net\\az140-42-msixvhds\\XmlNotepad.vhd**, 여기서 자리 표시자 `<storage-account-name>`은 **az140-42-msixvhds** 파일 공유를 호스팅하는 스토리지 계정의 이름을 나타냄|
   |MSIX 패키지|패키지를 만드는 동안 생성된 이름|
   |표시 이름|**XML 메모장**|
   |등록 유형|**주문형**|
   |State(상태)|**진행 중**|

#### 작업 4: 애플리케이션 그룹에 MSIX 앱 게시

> **참고**: 원격 앱과 데스크톱 앱 그룹 모두에 MSIX 앱을 게시합니다. 

1. **az140-cl-vm42**에 연결된 Bastion 내의 Azure Portal이 표시된 Microsoft Edge 창에서 **Azure Virtual Desktop**을 검색하여 선택합니다. 그런 다음 **Azure Virtual Desktop** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **애플리케이션 그룹**을 선택합니다.
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 블레이드에서 **az140-21-hp1-Utilities-RAG** 애플리케이션 그룹 항목을 선택합니다.
1. **az140-21-hp1-Utilities-RAG** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **애플리케이션**을 선택합니다. 
1. **az140-21-hp1-Utilities-RAG \| 애플리케이션** 블레이드에서 **+ 추가**를 클릭합니다.
1. **애플리케이션 추가** 블레이드의 **기본 사항** 및 **아이콘** 탭에서 다음 설정을 지정하고 **검토 + 추가**를 선택합니다.

   |설정|값|
   |---|---|
   |애플리케이션 소스|**앱 연결**|
   |Package(패키지)|이미지에 포함된 패키지를 나타내는 이름|
   |애플리케이션|**XMLNOTEPAD**|
   |애플리케이션 식별자|**XML 메모장**|
   |표시 이름|**XML 메모장**|
   |설명|**XML 메모장**|

1. 구성된 설정을 검토한 다음 **추가**를 선택합니다.
1. **Azure Virtual Desktop \| 애플리케이션 그룹** 블레이드로 다시 이동하여 **az140-21-hp1-DAG** 애플리케이션 그룹 항목을 선택합니다.
1. **az140-21-hp1-DAG** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **애플리케이션**을 선택합니다. 
1. **az140-21-hp1-DAG \| 애플리케이션** 블레이드에서 **+ 추가**를 클릭합니다.
1. **애플리케이션 추가** 블레이드에서 다음 설정을 지정하고 **검토 + 추가**를 선택합니다.

   |설정|값|
   |---|---|
   |애플리케이션 소스|**MSIX 패키지**|
   |MSIX 패키지|이미지에 포함된 패키지를 나타내는 이름|
   |애플리케이션 식별자|**XML 메모장**|
   |표시 이름|**XML 메모장**|
   |설명|**XML 메모장**|

1. 구성된 설정을 검토한 다음 **추가**를 선택합니다.

#### 작업 5: MSIX 앱 연결의 기능 유효성 검사

1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 Microsoft Edge를 시작하고 [Windows 데스크톱 클라이언트 다운로드 페이지](https://go.microsoft.com/fwlink/?linkid=2068602)로 이동한 후 다운로드가 완료되면 **파일 열기**를 선택하여 설치를 시작합니다. **원격 데스크톱 설정** 마법사의 **설치 범위** 페이지에서 **이 컴퓨터의 모든 사용자에 대해 설치** 옵션을 선택하고 **설치**를을 클릭합니다. 
1. 설치가 완료되면 **설정이 종료되면 원격 데스크톱 시작** 체크박스가 선택되어 있는지 확인한 후 **마침**을 클릭하여 원격 데스크톱 클라이언트를 시작합니다.
1. **Remote Desktop** 클라이언트 창에서 **구독**을 선택하고 메시지가 표시되면 **aduser1** 사용자 계정 이름으로 로그인합니다. 암호로는 이 사용자 계정을 만들 때 설정한 암호를 사용합니다. 
1. **원격 데스크톱** 클라이언트 창의 **az140-21-ws1** 섹션에서 **XML 메모장** 아이콘을 두 번 클릭합니다. 메시지가 표시되면 암호를 입력하고 XML Notepad가 정상적으로 시작되었는지 확인합니다.
1. **az140-cl-vm42**에 연결된 Bastion 세션 내에서 **시작**을 마우스 오른쪽 단추로 클릭하고 오른쪽 클릭 메뉴에서 **종료 또는 로그아웃**을 선택한 다음, 계단식 메뉴에서 **로그아웃**을 선택합니다.
1. **연결 끊김** 대화 상자에서 **닫기**를 선택합니다.

### 연습 4: 랩에서 프로비전 및 사용한 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전 및 사용한 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 해당 컴퓨팅 비용을 최소화하기 위해 이 랩에서 프로비전 및 사용한 Azure VM의 할당을 취소합니다.

#### 작업 1: 랩에서 프로비전 및 사용한 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 **Cloud Shell** 창 내에서 **PowerShell** 셸 세션을 엽니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM의 목록을 표시합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG'
   Get-AzVM -ResourceGroup 'az140-42-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에서 만들고 사용한 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-21-RG' | Stop-AzVM -NoWait -Force
   Get-AzVM -ResourceGroup 'az140-42-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 비동기적으로 실행되므로(-NoWait 매개 변수에 의해 결정됨) 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수는 있지만, Azure VM이 실제로 중지 및 할당 취소되려면 몇 분 정도 걸립니다.
