---
lab:
  title: '랩: PowerShell(AD DS)을 사용하여 호스트 풀 및 호스트 배포 및 관리'
  module: 'Module 2: Implement a WVD Infrastructure'
---

# 랩 - PowerShell을 사용하여 호스트 풀 및 호스트 배포 및 관리
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Azure AD 계정 및 해당 Azure 구독과 연결된 Azure AD 테넌트의 Global 관리istrator 역할
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩

## 예상 소요 시간

60분

## 실습 시나리오

AD DS(Active Directory 도메인 Services) 환경에서 PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트의 배포를 자동화해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 및 호스트 배포
- PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀에 호스트 추가

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json
-  \\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json

## 지침

### 연습 1: PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 및 세션 호스트 구현
  
이 연습의 주요 작업은 다음과 같습니다.

1. PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 배포 준비
1. PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 만들기
1. PowerShell을 사용하여 Windows 10 Enterprise를 실행하는 Azure VM의 템플릿 기반 배포 수행
1. PowerShell을 사용하여 Windows 10 Enterprise를 세션 호스트로 실행하는 Azure VM을 Azure Virtual Desktop 호스트 풀에 추가
1. Azure Virtual Desktop 세션 호스트의 배포 확인

#### 작업 1: PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 배포 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Windows PowerShell ISE**를 관리자로 시작**** 합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Azure Virtual Desktop 풀 세션 호스트의 컴퓨터 개체를 호스트할 WVDInfra**라는 **조직 단위의 고유 이름을 식별**합니다.

   ```powershell
   (Get-ADOrganizationalUnit -Filter "Name -eq 'WVDInfra'").distinguishedName
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Azure Virtual Desktop 호스트를 AD DS에 조인하는 데 사용할 ADATUM\\Student** 계정의 **UPN 접미사를 식별합니다기본(**student@adatum.com**).**

   ```powershell
   (Get-ADUser -Filter {sAMAccountName -eq 'student'} -Properties userPrincipalName).userPrincipalName
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 DesktopVirtualization PowerShell 모듈을 설치합니다(메시지가 표시되면 [모두** 예 클릭]****).

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -Force
   ```

   > **참고**: 사용 중인 기존 PowerShell 모듈에 대한 경고는 무시합니다.

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정의 Azure AD 자격 증명을 사용하여 로그인합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Azure Portal에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서** 텍스트 상자를 사용하여 **가상 네트워크를 검색 및 탐색**하고 가상 네트워크**** 블레이드에서 **az140-adds-vnet11을** 선택합니다**.** 
1. az140-adds-vnet11** 블레이드에서 서브넷을 선택하고 **, 서브넷** 블레이드에서 **+ 서브넷**을 선택하고 **, 서브넷**** 추가 블레이드에서 **다음 설정을 지정하고(다른 모든 설정을 기본값으로 유지) 저장**을 클릭합니다**.**

   |설정|값|
   |---|---|
   |속성|**hp3-Subnet**|
   |서브넷 주소 범위|**10.0.3.0/24**|

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서** 텍스트 상자를 사용하여 **네트워크 보안 그룹을 검색하고 탐색**하고 네트워크 **보안 그룹**** 블레이드에서 **az140-11-RG** 리소스 그룹에서 보안 그룹을** 선택합니다.
1. 네트워크 보안 그룹 블레이드의 왼쪽 **세로 메뉴에서 설정** 섹션에서 속성을** 클릭합니다**.
1. 속성** 블레이드의 **리소스 ID** 텍스트 상자 오른쪽에 있는 클립보드**로 복사 아이콘을 **클릭합니다**. 

   > **참고**: 구독 ID는 다르지만 값은 형식 `/subscriptions/de8279a3-0675-40e6-91e2-5c3728792cb5/resourceGroups/az140-11-RG/providers/Microsoft.Network/networkSecurityGroups/az140-cl-vm11-nsg`과 유사해야 합니다. 다음 작업에서 필요하므로 기록합니다.

#### 작업 2: PowerShell을 사용하여 Azure Virtual Desktop 호스트 풀 만들기

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Azure 구독에 로그인**합니다.

   ```powershell
   Connect-AzAccount
   ```

1. 메시지가 표시되면 사용자 계정의 자격 증명에 이 랩에서 사용 중인 구독의 소유자 역할을 제공합니다.
1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Azure 가상 네트워크 **az140-adds-vnet111**을 호스팅하는 Azure 지역을 식별**합니다.

   ```powershell
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 호스트 풀 및 해당 리소스를 호스트할 리소스 그룹을 만듭니**다.

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 빈 호스트 풀을 만듭니**다.

   ```powershell
   $hostPoolName = 'az140-24-hp3'
   $workspaceName = 'az140-24-ws1'
   $dagAppGroupName = "$hostPoolName-DAG"
   New-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -WorkspaceName $workspaceName -HostPoolType Pooled -LoadBalancerType BreadthFirst -Location $location -DesktopAppGroupName $dagAppGroupName -PreferredAppGroupType Desktop 
   ```

   > **참고**: **New-AzWvdHostPool** cmdlet을 사용하면 호스트 풀, 작업 영역 및 데스크톱 앱 그룹을 만들고 데스크톱 앱 그룹을 작업 영역에 등록할 수 있습니다. 새 작업 영역을 만들거나 기존 작업 영역을 사용할 수 있습니다.

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 az140-wvd-pooled**라는 **Azure AD 그룹의 objectID 특성을 검색**합니다.

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-pooled').Id
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 az140-wvd-pooled**라는 **Azure AD 그룹을 새로 만든 호스트 풀의 기본 데스크톱 앱 그룹에 할당**합니다.

   ```powershell
   $roleDefinitionName = 'Desktop Virtualization User'
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName $roleDefinitionName -ResourceName $dagAppGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

#### 작업 3: PowerShell을 사용하여 Windows 10 Enterprise를 실행하는 Azure VM의 템플릿 기반 배포 수행

1. 랩 컴퓨터에서 배포된 스토리지 계정으로 이동합니다. 파일 공유 블레이드에서 **az140-22-profiles** 파일 공유를 선택합니다.

1. **업로드**를 선택하고 랩 파일 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\02\\az140-24_azuredeployhp3.parameters.json**을 모두 파일 공유에 업로드합니다.

1. **az140-dc-vm11**에 대한 베스천 세션 내에서 파일 탐색기 열고 이전에 구성된 Z: 또는 파일 공유 연결에 할당된 드라이브 문자로 이동합니다. 업로드된 배포 파일을 **C:\AllFiles\Labs\02**에 복사합니다.

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이전 작업에서 **만든 호스트 풀에서 Azure Virtual Desktop 세션 호스트 역할을 하는 Windows 10 Enterprise(다중 세션)를 실행하는 Azure VM을 배포**합니다.

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab24hp3Deployment `
     -TemplateFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.json `
     -TemplateParameterFile C:\AllFiles\Labs\02\az140-24_azuredeployhp3.parameters.json
   ```

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 작업을 진행하세요. 5분 정도 걸릴 수 있습니다. 

   > **참고**: 배포는 Azure Resource Manager 템플릿을 사용하여 Azure VM을 프로비전하고 운영 체제를 AD DS가 수행하는 adatum.com** 자동으로 조인하는 **VM 확장을 적용합니다기본.

1. az140-dc-vm11**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 세 번째 세션 호스트가 adatum.com** AD DS에 성공적으로 조인되었는지 **확인합니다기본

   ```powershell
   Get-ADComputer -Filter "sAMAccountName -eq 'az140-24-p3-0$'"
   ```

#### 작업 4: PowerShell을 사용하여 Windows 10 Enterprise를 호스트로 실행하는 Azure VM을 Azure Virtual Desktop 호스트 풀에 추가

1. az140-dc-vm11에 **대한 Bastion 세션 내에서 Azure Portal을 표시하는 브라우저 창에서 가상 머신을 검색하여 선택하고 **가상 **머신** 블레이드의 가상 머신** 목록에서 az140-24-p3-0**을 선택합니다**.**
1. **az140-24-p3-0** 블레이드에서 **연결**을 선택하고, 드롭다운 메뉴에서 **RDP**를 선택하고, **az140-24-p3-0 \| 연결** 블레이드의 **RDP** 탭에 있는 **IP 주소** 드롭다운 목록에서 **개인 IP 주소(10.0.3.4)** 항목을 선택한 다음, **RDP 파일 다운로드**를 선택합니다.
1. 메시지가 표시되면 다음 자격 증명으로 로그인합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\\Student**|
   |암호|**Pa55w.rd1234**|

1. az140-24-p3-0에 대한 **원격 데스크톱 세션 내에서 Windows PowerShell ISE**를 관리자 권한으로 시작**** 합니다.
1. az140-24-p3-0에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 새로 배포된 Azure VM을 세션 호스트로 이 랩의 앞부분에서 프로비전한 호스트 풀에 추가하는 데 필요한 파일을 호스트하는 폴더를 만듭니**다.

   ```powershell
   $labFilesFolder = 'C:\AllFiles\Labs\02'
   New-Item -ItemType Directory -Path $labFilesFolder
   ```

   >**참고** [T] 구문을 사용하여 PowerShell cmdlet을 복사합니다. 경우에 따라 복사된 텍스트가 잘못될 수 있습니다(예: $sign이 4 숫자 문자로 표시됨). cmdlet을 실행하기 전에 이를 수정해야 합니다. PowerShell ISE **스크립트** 창으로 복사하고 수정한 다음, 수정된 텍스트를 강조 표시하고 **F8** 키를 누릅니다(**선택 영역 실행**).

1. az140-24-p3-0에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 호스트 풀에 세션 호스트를 추가하는 데 필요한 Azure Virtual Desktop 에이전트 및 부팅 로더 설치 관리자를 다운로드**합니다.

   ```powershell
   $webClient = New-Object System.Net.WebClient
   $wvdAgentInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrmXv'
   $wvdAgentInstallerName = 'WVD-Agent.msi'
   $webClient.DownloadFile($wvdAgentInstallerURL,"$labFilesFolder/$wvdAgentInstallerName")
   $wvdBootLoaderInstallerURL = 'https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RWrxrH'
   $wvdBootLoaderInstallerName = 'WVD-BootLoader.msi'
   $webClient.DownloadFile($wvdBootLoaderInstallerURL,"$labFilesFolder/$wvdBootLoaderInstallerName")
   ```

1. az140-24-p3-0에 대한 **원격 데스크톱 세션 내에서 관리이스트레이터: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 최신 버전의 PowerShellGet 모듈을 설치합니다(확인 메시지가 표시되면 예** 선택**).**

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 최신 버전의 Az.DesktopVirtualization PowerShell 모듈을 설치합니다.

   ```powershell
   Install-Module -Name Az.DesktopVirtualization -AllowClobber -Force
   Install-Module -Name Az -AllowClobber -Force
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 PowerShell 실행 정책을 수정하고 Azure 구독에 로그인합니다.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope CurrentUser -Force
   Connect-AzAccount
   ```

1. 메시지가 표시되면 사용자 계정의 자격 증명에 이 랩에서 사용 중인 구독의 소유자 역할을 제공합니다.
1. az140-24-p3-0에 대한 **원격 Desktopliveid 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이 연습의 앞부분에서 **프로비전한 풀에 새 세션 호스트를 조인하는 데 필요한 토큰을 생성**합니다.

   ```powershell
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $registrationInfo = New-AzWvdRegistrationInfo -ResourceGroupName $resourceGroupName -HostPoolName $hostPoolName -ExpirationTime $((get-date).ToUniversalTime().AddDays(1).ToString('yyyy-MM-ddTHH:mm:ss.fffffffZ'))
   ```
   > **참고**: 세션 호스트가 호스트 풀에 조인하도록 권한을 부여하려면 등록 토큰이 필요합니다. 토큰의 만료 날짜 값은 현재 날짜와 시간으로부터 1시간에서 1개월 사이여야 합니다.

1. az140-24-p3-0**에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Azure Virtual Desktop 에이전트를 설치합니다.

   ```powershell
   Set-Location -Path $labFilesFolder
   Start-Process -FilePath 'msiexec.exe' -ArgumentList "/i $WVDAgentInstallerName", "/quiet", "/qn", "/norestart", "/passive", "REGISTRATIONTOKEN=$($registrationInfo.Token)", "/l* $labFilesFolder\AgentInstall.log" | Wait-Process
   ```

1. az140-24-p3-0**에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Azure Virtual Desktop 부팅 로더를 설치합니다.

   ```powershell
   Start-Process -FilePath "msiexec.exe" -ArgumentList "/i $wvdBootLoaderInstallerName", "/quiet", "/qn", "/norestart", "/passive", "/l* $labFilesFolder\BootLoaderInstall.log" | Wait-process
   ```

#### 작업 5: Azure Virtual Desktop 호스트의 배포 확인

1. 랩 컴퓨터로 돌아간 후 Azure Portal이 표시된 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 블레이드에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 블레이드에서 새로 수정된 풀을 나타내는 **az140-24-hp3** 항목을 선택합니다.
1. **az140-24-hp3** 블레이드 왼쪽의 세로 메뉴에 있는 **관리** 섹션에서 **세션 호스트**를 선택합니다. 
1. **az140-24-hp3 \| 세션 호스트** 블레이드에서 배포에 호스트 하나가 포함되어 있는지 확인합니다.

#### 작업 6: PowerShell을 사용하여 앱 그룹 관리

1. 랩 컴퓨터에서 Bastion 세션으로 전환하여 **az140-dc-vm11**로 전환하고 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 원격 앱 그룹을 만듭니다.

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $appGroupName = 'az140-24-hp3-Office365-RAG'
   $resourceGroupName = 'az140-24-RG'
   $hostPoolName = 'az140-24-hp3'
   $location = (Get-AzVirtualNetwork -ResourceGroupName 'az140-11-RG' -Name 'az140-adds-vnet11').Location
   New-AzWvdApplicationGroup -Name $appGroupName -ResourceGroupName $resourceGroupName -ApplicationGroupType 'RemoteApp' -HostPoolArmPath "/subscriptions/$subscriptionId/resourcegroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName"-Location $location
   ```

1. az140-dc-vm11에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 풀 호스트의 시작** 메뉴 앱을 나열**하고 출력을 검토**합니다.

   ```powershell
   Get-AzWvdStartMenuItem -ApplicationGroupName $appGroupName -ResourceGroupName $resourceGroupName | Format-List | more
   ```

   > **참고**: 게시하려는 애플리케이션의 경우 FilePath, IconPath****** 및 IconIndex**와 **같은 **매개 변수를 포함하여 출력에 포함된 정보를 기록해야 합니다.

1. az140-dc-vm11**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Microsoft Word를 게시합니다.

   ```powershell
   $name = 'Microsoft Word'
   $filePath = 'C:\Program Files\Microsoft Office\root\Office16\WINWORD.EXE'
   $iconPath = 'C:\Program Files\Microsoft Office\Root\VFS\Windows\Installer\{90160000-000F-0000-1000-0000000FF1CE}\wordicon.exe'
   New-AzWvdApplication -GroupName $appGroupName -Name $name -ResourceGroupName $resourceGroupName -FriendlyName $name -Filepath $filePath -IconPath $iconPath -IconIndex 0 -CommandLineSetting 'DoNotAllow' -ShowInPortal:$true
   ```

1. az140-dc-vm11**에 대한 **Bastion 세션 내에서 관리istrator: Windows PowerShell ISE** 콘솔에서 **다음을 실행하여 Microsoft Word를 게시합니다.

   ```powershell
   $aadGroupObjectId = (Get-AzADGroup -DisplayName 'az140-wvd-remote-app').Id
   New-AzRoleAssignment -ObjectId $aadGroupObjectId -RoleDefinitionName 'Desktop Virtualization User' -ResourceName $appGroupName -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
   ```

1. 랩 컴퓨터로 전환하고, Azure Portal을 표시하는 웹 브라우저의 **az140-24-hp3 \| 세션 호스트** 블레이드의 왼쪽 세로 메뉴에 있는 **관리** 섹션의 **애플리케이션 그룹**을 선택합니다.
1. **az140-24-hp3 \| 애플리케이션 그룹** 블레이드의 애플리케이션 그룹 목록에서 **az140-24-hp3-Office365-RAG** 항목을 선택합니다.
1. **az140-24-hp3-Office365-RAG** 블레이드에서 애플리케이션 및 할당을 포함하여 애플리케이션 그룹의 구성을 확인합니다.

### 연습 2: 랩에서 프로비전된 Azure VM 중지 및 할당 취소

이 연습의 주요 작업은 다음과 같습니다.

1. 랩에서 프로비전된 Azure VM 중지 및 할당 취소

>**참고**: 이 연습에서는 이 랩에 프로비전된 Azure VM의 할당을 취소하여 해당 컴퓨팅 요금을 최소화합니다.

#### 작업 1: 랩에서 프로비전된 Azure VM 할당 취소

1. 랩 컴퓨터로 전환하고 Azure Portal을 표시하는 웹 브라우저 창에서 Cloud Shell 창 내에서 **PowerShell** 셸** 세션을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 나열합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG'
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이 랩에서 만든 모든 Azure VM을 중지하고 할당을 취소합니다.

   ```powershell
   Get-AzVM -ResourceGroup 'az140-24-RG' | Stop-AzVM -NoWait -Force
   ```

   >**참고**: 명령은 -NoWait 매개 변수에 의해 결정된 대로 비동기적으로 실행되므로 동일한 PowerShell 세션 내에서 즉시 다른 PowerShell 명령을 실행할 수 있지만 Azure VM이 실제로 중지되고 할당이 취소되기까지 몇 분 정도 걸립니다.
