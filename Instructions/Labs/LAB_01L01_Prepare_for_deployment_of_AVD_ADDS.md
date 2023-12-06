---
lab:
  title: '랩: AD DS(Azure Virtual Desktop) 배포 준비'
  module: 'Module 1: Plan a AVD Architecture'
---

# 랩 - AD DS(Azure Virtual Desktop) 배포 준비
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할

## 예상 소요 시간

60분

## 랩 시나리오

AD DS(Active Directory 도메인 Services) 환경 배포를 준비해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure VM을 사용하여 AD DS(Active Directory 도메인 Services) single-do기본 포리스트 배포
- Microsoft Entra 테넌트에 AD DS 포리스트 통합

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## 지침

### 연습 0: vCPU 할당량 수 늘리기

이 연습의 주요 작업은 다음과 같습니다.

1. 현재 vCPU 사용량 식별
1. vCPU 할당량 증가 요청

#### 작업 1: 현재 vCPU 사용량 식별

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. 등록되지 않은 경우, Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 **Microsoft.Compute** 및 **Microsoft.Network** 리소스 공급자의 등록 상태를 확인합니다.

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. Azure Portal의 Cloud Shell PowerShell 세션에서 **다음을 실행하여 Microsoft.Compute** 리소스 공급자의 **등록 상태 확인합니다.**

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**참고**: 상태 등록됨으로 **나열되었는지** 확인합니다. 그렇지 않은 경우 몇 분 정도 기다렸다가 이 단계를 반복합니다.

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음 명령을 실행하여 다음 명령의 위치를 설정합니다(자리 표시자를 이 랩에 사용하려는 Azure 지역 이름으로 바꿉 `<Azure_region>` 니다(예: 예: `eastus`).

   ```powershell
   $location = '<Azure_region>'
   ```

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 vCPU의 현재 사용량과 StandardDSv3Family 및 **StandardBSFamily**** Azure VM에 대한 **해당 제한을 식별합니다. 

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

   > **참고**: Azure 지역의 이름을 식별하려면 Cloud Shell**의 **PowerShell 프롬프트에서 실행`(Get-AzLocation).Location`합니다.
   
1. 이전 단계에서 실행된 명령의 출력을 검토하고 대상 Azure 지역에 있는 Azure VM의 표준 DSv3 제품군 vCPU에 **30**개 이상의 **사용 가능한 vCPU**가 있는지 확인합니다. 이미 해당되는 경우 다음 연습으로 직접 진행합니다. 그렇지 않으면 이 연습의 다음 작업을 진행합니다. 

#### 작업 2: vCPU 할당량 증가 요청

1. Azure Portal에서 구독**을 검색하여 선택하고 **구독 블레이드에서 **** 이 랩에 사용하려는 Azure 구독을 나타내는 항목을 선택합니다.
1. Azure Portal의 구독 블레이드 왼쪽에 있는 세로 **메뉴의 설정** 섹션에서 사용량 + 할당량을** 선택합니다**. 

   **참고:** 할당량을 늘리기 위해 지원 티켓을 올릴 필요가 없을 수도 있습니다.

   **참고:** 할당량 증가를 요청하려면 MFA(다단계 인증)를 사용하여 로그인해야 합니다. MFA를 사용하여 계정을 구성해야 하는 경우 [Azure Active Directory Multi-Factor Authentication 배포 계획](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted)을 참조하세요. 
   
1. **Azure Pass – 스폰서쉽 | 사용량 + 할당량** 블레이드, 지역을** 선택하고**, 드롭다운 목록에서 이 랩에 사용할 Azure 지역의 이름 옆에 있는 검사 상자를 선택하고, 적용**을 선택하고**, 지역** 항목 왼쪽의 **드롭다운 목록에 컴퓨팅** 항목이 표시되는지 확인하고**, 검색 상자에 표준 DSv3**을 입력**합니다. 
1. 결과 목록에서 표준 DSv3 Family vCPU 항목 옆에 있는 **검사 상자를 선택하고 도구 모음에서 할당량 증가** 요청 항목을 선택한 **다음 드롭다운 목록에서 새 제한** 입력을 선택합니다**.**
1. **요청 할당량 증가** 창의 **새 제한** 열 텍스트 상자에 **30**을 입력한 다음, **제출**을 선택합니다.
1. 메시지가 표시되면 **할당량 증가 요청** 창에서 **다중 팩터리 인증으로 인증**을 선택하고 프롬프트에 따라 인증합니다.
1. 할당량 요청이 완료되도록 허용합니다.  잠시 후 **할당량 세부 정보** 블레이드에서 요청이 승인되고 할당량이 증가했음을 지정합니다. **할당량 세부 정보** 블레이드를 닫습니다.

   >**참고**: Azure 지역 선택 및 현재 수요에 따라 지원 요청을 발생시키는 것이 필요할 수 있습니다. 지원 요청을 만드는 프로세스에 대한 지침은 Azure 지원 요청[ 만들기를 참조](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)하세요.

### 연습 1: AD DS(Active Directory 도메인 서비스)를 배포합니다기본

이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM 배포 준비
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 AD DS를 실행하는 Azure VM을 배포합니다기본 컨트롤러
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. Azure Bastion 배포

#### 작업 1: Azure VM 배포 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal을 표시하는 웹 브라우저에서 Microsoft Entra 테넌트 개요** 블레이드로 **이동하고 왼쪽**의 세로 메뉴에서 [관리**] 섹션에서 [속성]**을 클릭합니다**.
1. **Microsoft Entra 테넌트 속성** 블레이드의 블레이드 맨 아래에 있는 보안 기본값** 관리 링크를 선택합니다**.
1. 보안 기본값 사용 블레이드에서 **필요한 경우 아니요**를 선택하고 **내 조직에서 조건부 액세스** 검사 상자를 사용하고 있는지 선택하고 **저장**을 선택합니다**.**
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 


#### 작업 2: Azure Resource Manager 빠른 시작 템플릿을 사용하여 AD DS를 실행하는 Azure VM 배포기본 컨트롤러

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure_region>` 자리 표시자를 이 랩에 사용할 Azure 지역 이름으로 바꿉니다(예: `eastus`)).

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Azure Portal에서 Cloud Shell** 창을 닫습니다**.
1. 랩 컴퓨터의 동일한 웹 브라우저 창에서 다른 웹 브라우저 탭을 열고 [새 Windows VM 만들기 및 새 AD 포리스트, 도메인, DC 만들기](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)라는 빠른 시작 템플릿의 사용자 지정 버전을 탐색합니다. 
1. **새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기** 페이지에서 **Azure에 배포**를 선택합니다. 이렇게 하면 Azure Portal에 **Create an Azure VM with a new AD Forest** 블레이드로 브라우저가 자동으로 리디렉션됩니다.
1. **새 AD 포리스트** 블레이드를 사용하여 Azure VM 만들기에서 매개 변수** 편집을 선택합니다**.
1. **매개 변수 편집** 블레이드의 **열기** 대화 상자에서 **파일 로드**를 선택하고 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**을 선택한 후에 **열기**, **저장**을 차례로 선택합니다. 
1. **새 AD 포리스트를 사용하여 Azure VM 만들기** 블레이드에서 다음 설정을 지정합니다(나머지는 기존 값을 그대로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-11-RG**|
   |도메인 이름|**adatum.com**|

1. **새 AD 포리스트**를 사용하여 Azure VM 만들기 블레이드에서 검토 + 만들기를 선택하고 **만들기****를 선택합니다**.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 15분 정도 걸릴 수 있습니다. 

#### 작업 3: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

1. 랩 컴퓨터에서 Azure Portal이 표시된 웹 브라우저에서 Cloud Shell 창의 PowerShell 세션을 열고 다음 명령을 실행하여 이전 작업에서 만든 **az140-adds-vnet11** 가상 네트워크에 **cl-Subnet** 서브넷을 추가합니다.

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Azure Portal의 Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택합니다. 그런 다음, **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 클라이언트 역할을 할 Windows 10을 실행하는 Azure VM을 새로 만든 서브넷에 배포합니다.

   ```powershell
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11.parameters.json
   ```

   > **참고**: 배포가 완료될 때까지 기다리지 말고 다음 작업을 진행하세요. 배포는 10분 정도 걸릴 수 있습니다.

#### 작업 4: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

> **참고**: 브라우저에 팝업 기능이 사용하도록 설정되어 있는지 확인합니다.

1. Azure Portal을 표시하는 브라우저 창에서 다른 탭을 열고 브라우저 탭에서 Azure Portal[로 ](https://portal.azure.com)이동합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 **AzureBastionSubnet**이라는 서브넷을 이 연습의 앞부분에서 만든 가상 네트워크 **az140-adds-vnet11**에 추가합니다.

   ```powershell
   $resourceGroupName = 'az140-11-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-adds-vnet11'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell 창을 닫습니다.
1. Azure Portal에서 **베스천**을 검색하여 선택하고 **베스천** 블레이드에서 **+ 만들기**를 선택합니다.
1. **베스천 만들기** 블레이드의 **기본 사항** 탭에서 다음 설정을 지정하고 **검토 + 만들기**를 선택합니다.

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-11-RG**|
   |이름|**az140-11-bastion**|
   |지역|이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역|
   |계층|**기본**|
   |가상 네트워크|**az140-adds-vnet11**|
   |서브넷|**AzureBastionSubnet(10.0.254.0/24)**|
   |공용 IP 주소|**새로 만들기**|
   |공용 IP 이름|**az140-adds-vnet11-ip**|

1. **베스천 만들기** 블레이드의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 배포는 5분 정도 걸릴 수 있습니다.

### 연습 2: Microsoft Entra 테넌트에 AD DS 포리스트 통합
  
이 연습의 주요 작업은 다음과 같습니다.

1. Microsoft Entra에 동기화될 AD DS 사용자 및 그룹 만들기
1. AD DS UPN 접미사 구성
1. Microsoft Entra와의 동기화를 구성하는 데 사용할 Microsoft Entra 사용자 만들기
1. Microsoft Entra Connect 설치
1. 하이브리드 Microsoft Entra 조인 구성

#### 작업 1: Microsoft Entra에 동기화될 AD DS 사용자 및 그룹 만들기

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Windows PowerShell ISE**를 관리자 권한으로 시작**** 합니다.
1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 관리istrators에 대해 Internet Explorer 보안 강화를 사용하지 않도록 설정합니다.

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이 랩에서 사용되는 Microsoft Entra 테넌트에 대한 동기화 범위에 포함된 개체를 포함하는 AD DS 조직 구성 단위를 만듭니다.

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Windows 10 do기본 조인 클라이언트 컴퓨터의 컴퓨터 개체를 포함하는 AD DS 조직 구성 단위를 만듭니다.

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩에서 사용되는 Microsoft Entra 테넌트에 동기화되는 AD DS 사용자 계정을 만듭니다(두 `<password>` 자리 표시자를 임의의 복잡한 암호로 대체).

   > **참고**: 사용된 암호를 기록해야 합니다. 이 랩 및 후속 랩의 뒷부분에서 필요합니다.

   ```powershell
   $ouName = 'ToSync'
   $ouPath = "OU=$ouName,DC=adatum,DC=com"
   $adUserNamePrefix = 'aduser'
   $adUPNSuffix = 'adatum.com'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AdUser -Name $adUserNamePrefix$counter -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix$counter@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru
   } 

   $adUserNamePrefix = 'wvdadmin1'
   $adUPNSuffix = 'adatum.com'
   New-AdUser -Name $adUserNamePrefix -Path $ouPath -Enabled $True `
       -ChangePasswordAtLogon $false -userPrincipalName $adUserNamePrefix@$adUPNSuffix `
       -AccountPassword (ConvertTo-SecureString '<password>' -AsPlainText -Force) -passThru

   Get-ADGroup -Identity 'Domain Admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

   > **참고**: 스크립트는 aduser1 aduser9라는 **9**개의 권한 없는 사용자 계정과 wvdadmin1** - ****이라는 **ADATUM\\Do기본 관리s** 그룹의 멤버**인 하나의 권한 있는 계정을 만듭니다.

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 랩에서 사용되는 Microsoft Entra 테넌트에 동기화될 AD DS 그룹 개체를 만듭니다.

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 이전 단계에서 만든 그룹에 구성원을 추가합니다.

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### 작업 2: AD DS UPN 접미사 구성

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 최신 버전의 PowerShellGet 모듈을 설치합니다(확인 메시지가 표시되면 예** 선택**).**

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 최신 버전의 Az PowerShell 모듈을 설치합니다(확인 메시지가 표시되면 모두 예** 선택**).

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독에 로그인합니다.

   ```powershell
   Connect-AzAccount
   ```

1. 메시지가 표시되면 사용자 계정의 자격 증명에 이 랩에서 사용 중인 구독의 소유자 역할을 제공합니다.
1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트의 ID 속성을 검색합니다.

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 최신 버전의 Azure AD PowerShell 모듈을 설치하고 가져옵니다.

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Microsoft Entra 테넌트에 인증합니다.

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. 메시지가 표시되면 이 작업의 앞부분에서 사용한 것과 동일한 자격 증명(이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정)으로 로그인합니다. 
1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트의 기본 DNS do기본 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트의 기본 DNS do기본 이름을 AD DS 포리스트의 UPN 접미사 목록에 추가합니다.

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 AD DS의 모든 사용자의 UPN 접미사로 Azure 구독과 연결된 Microsoft Entra 테넌트의 기본 DNS do기본 이름을 할당합니다기본

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 adatum.com** UPN 접미사를 **Student** do기본 사용자에게 할당**합니다.

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### 작업 3: 디렉터리 동기화를 구성하는 데 사용할 Microsoft Entra 사용자 만들기

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 새 Microsoft Entra 사용자를 만듭니다(자리 표시자를 임의의 복잡한 암호로 대체`<password>`**).

   > **참고**: 사용한 암호를 기록해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 새로 만든 Microsoft Entra 사용자에게 Global 관리istrator 역할을 할당합니다. 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 새로 만든 Microsoft Entra 사용자의 사용자 계정 이름을 식별합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **참고**: 사용자 계정 이름을 기록합니다. 나중에 이 연습에서 필요할 것입니다. 


#### 작업 4: Microsoft Entra 커넥트 설치

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 TLS 1.2를 해제**합니다.

   ```powershell
   New-Item 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SystemDefaultTlsVersions' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
   New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
   Write-Host 'TLS 1.2 has been enabled.'
   ```
   
1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Internet Explorer를 시작하고 비즈니스용 Microsoft Edge 다운로드 페이지[로 ](https://www.microsoft.com/en-us/edge/business/download)이동합니다.**
1. [비즈니스용 Microsoft Edge 다운로드 페이지에서](https://www.microsoft.com/en-us/edge/business/download) 안정적인 최신 버전의 Microsoft Edge를 다운로드하고, 설치하고, 시작하고, 기본 설정으로 구성합니다.
1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Microsoft Edge를 사용하여 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. Azure Portal에서 Azure Portal 페이지의 맨 위에 있는 검색 리소스, 서비스 및 문서 텍스트 상자를 사용하여 **Azure Active Directory** 블레이드를 검색하고 탐색**하고, Microsoft Entra 테넌트 블레이드**의 허브 메뉴의 관리** 섹션에서 Microsoft Entra 커넥트** 선택합니다**.**
1. **Microsoft Entra 커넥트** 블레이드에서 먼저 왼쪽의 **커넥트 동기화** 링크를 선택한 다음 Microsoft Entra 커넥트** 다운로드 링크를 선택합니다**. 그러면 Microsoft Azure Active Directory 커넥트** 다운로드 페이지가 **표시되는 새 브라우저 탭이 자동으로 열립니다.
1. **Microsoft Azure Active Directory 커넥트** 다운로드 페이지에서 다운로드**를 선택합니다**.
1. **AzureADConnect.msi** 설치 관리자를 실행할지 아니면 저장할지 묻는 메시지가 표시되면 **실행**을 선택합니다. 그렇지 않으면 다운로드한 후 파일을 열어 **Microsoft Azure Active Directory 연결** 마법사를 시작합니다.
1. **Microsoft Azure Active Directory 커넥트** 마법사의 **Microsoft Entra 커넥트** 시작 페이지에서 사용 조건 및 개인 정보 보호 알림**에 동의하는 검사 상자를 **선택하고 계속**을 선택합니다**.
1. **Microsoft Azure Active Directory 커넥트** 마법사의 **Express 설정** 페이지에서 사용자 지정** 옵션을 선택합니다**.
1. **필수 구성 요소 설치** 페이지에서 모든 선택적 구성 옵션의 선택을 취소하고 **설치**를 선택합니다.
1. 사용자 로그인** 페이지에서 암호 해시 동기화**만 **사용하도록 설정되어 있는지 확인하고 다음**을 선택합니다**.**
1. **Microsoft Entra**에 대한 커넥트 페이지에서 이전 연습에서 만든 aadsyncuser** 사용자 계정의 **자격 증명을 사용하여 인증하고 다음**을 선택합니다**. 

   > **참고**: 이 연습의 앞부분에서 기록한 aadsyncuser** 계정의 **userPrincipalName 특성을 제공하고 이 랩의 앞부분에서 설정한 암호를 암호로 지정합니다.

1. **디렉터리** 커넥트 페이지에서 adatum.com** 포리스트 항목의 **오른쪽에 있는 디렉터리** 추가 단추를 선택합니다**.
1. **AD 포리스트 계정** 창에서 새 AD 계정** 만들기 옵션이 **선택되어 있는지 확인하고, 다음 자격 증명을 지정하고, 확인을** 선택합니다**.

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\Student**|
   |암호|**Pa55w.rd1234**|

1. 디렉터리** 커넥트 페이지로 돌아가서 adatum.com** 항목이 **구성된 디렉터리로 표시되는지 확인하고 다음을 선택합니다 **.****
1. **Microsoft Entra 로그인 구성** 페이지에서 UPN 접미사가 확인된 do기본 이름과** 일치하지 않는 경우 사용자가 온-프레미스 자격 증명으로 Microsoft Entra에 로그인할 수 없다는 경고**가 표시되고, 확인될 모든 UPN 접미사를 일치시키지 않고 검사box **Continue를 사용하도록 설정하고기본** 다음**을 선택합니다**.

   > **참고**: Microsoft Entra 테넌트에 확인된 사용자 지정 DNS가 없으므로기본 adatum.com** AD DS의 **UPN 접미사 중 하나와 일치해야 합니다.

1. **Do기본 및 OU 필터링** 페이지에서 선택한 do기본 및 OU** 옵션을 **선택하고, adatum.com 노드를 확장하고, 모든 검사 상자의 선택을 취소하고, ToSync** OU 옆에 있는 **검사 상자만 선택하고, 다음**을 선택합니다**.
1. **고유하게 사용자** 식별 페이지에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **사용자 및 디바이스** 필터 페이지에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **선택적 기능** 페이지에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. 구성 준비 완료 페이지에서 구성이 완료**될 **때 동기화 프로세스 시작 검사 상자가 선택되어 있는지 확인하고 설치**를 선택합니다**.** ** 

   > **참고**: 설치에는 약 2분이 소요됩니다.

1. 구성 완료** 페이지에서 정보를 검토하고 종료**를 **선택하여 **Microsoft Azure Active Directory 커넥트** 창을 닫습니다**.
1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Adatum Lab Microsoft Entra 테넌트의 사용자 - 모든 사용자** 블레이드로 이동합니다**.**
1. **사용자 \| 모든 사용자** 블레이드에서 사용자 개체 목록에 이 랩 앞부분에서 만든 AD DS 사용자 계정 목록이 포함되어 있으며, **온-프레미스 동기화 사용 설정됨** 열에 **예** 항목이 표시되어 있음을 확인합니다.

   > **참고**: AD DS 사용자 계정이 표시될 때까지 몇 분 정도 기다렸다가 브라우저 페이지를 새로 고쳐야 할 수 있습니다.
