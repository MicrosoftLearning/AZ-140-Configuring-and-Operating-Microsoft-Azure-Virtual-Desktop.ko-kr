---
lab:
  title: '랩: Azure Virtual Desktop 배포 준비(AD DS)'
  module: 'Module 1: Plan a AVD Architecture'
---

# <a name="lab---prepare-for-deployment-of-azure-virtual-desktop-ad-ds"></a>랩 - Azure Virtual Desktop의 배포 준비(AD DS)
# <a name="student-lab-manual"></a>학생용 랩 매뉴얼

## <a name="lab-dependencies"></a>랩 종속성

- 이 랩에서 사용할 Azure 구독
- 이 랩에서 사용할 Azure 구독에 대한 Owner 또는 Contributor 역할, 그리고 해당 Azure 구독에 연결된 Azure AD 테넌트의 전역 관리자 역할이 할당되어 있는 Microsoft 계정 또는 Azure AD 계정

## <a name="estimated-time"></a>예상 소요 시간

60분

## <a name="lab-scenario"></a>랩 시나리오

AD DS(Active Directory Domain Services) 환경에서 배포를 준비해야 합니다.

## <a name="objectives"></a>목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure VM을 사용하여 Active Directory Domain Services(AD DS) 단일 도메인 포리스트 배포
- Azure AD 포리스트와 Azure Active Directory(Azure AD) 테넌트 통합

## <a name="lab-files"></a>랩 파일 

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11.parameters.json

## <a name="instructions"></a>Instructions

### <a name="exercise-0-increase-the-number-of-vcpu-quotas"></a>연습 0: vCPU 할당량 늘리기

이 연습의 주요 작업은 다음과 같습니다.

1. 현재 vCPU 사용량 파악
1. vCPU 할당량 늘리기 요청

#### <a name="task-1-identify-current-vcpu-usage"></a>작업 1: 현재 vCPU 사용량 파악

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 검색 텍스트 상자 바로 오른쪽의 도구 모음 아이콘을 선택하여 **Cloud Shell** 창을 엽니다.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. 등록되지 않은 경우, Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 **Microsoft.Compute** 및 **Microsoft.Network** 리소스 공급자의 등록 상태를 확인합니다.

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Network'
   ```

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 **Microsoft.Compute** 리소스 공급자의 등록 상태를 확인합니다.

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**참고**: 상태가 **등록됨**로 나와 있는지 확인합니다. 그렇지 않은 경우 몇 분 기다렸다가 이 단계를 반복합니다.

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음 명령을 실행하여 현재 vCPU 사용량, 그리고 **StandardDSv3Family** 및 **StandardBSFamily** Azure VM의 vCPU 사용량 한도를 확인합니다(`<Azure_region>` 자리 표시자는 `eastus` 등 이 랩에서 사용하려는 Azure 지역의 이름으로 바꿔야 함).

   ```powershell
   $location = '<Azure_region>'
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardBSFamily'}
   ```

   > **참고**: Azure 지역의 이름을 확인하려면 **Cloud Shell**의 PowerShell 프롬프트에서 `(Get-AzLocation).Location`을 실행합니다.
   
1. 이전 단계에서 실행한 명령 출력을 검토하여 대상 Azure 지역에서 Azure VM의 **Standard DSv3 Family vCPUs** 및 **Standard BS Family vCPUs** 둘 다에서 사용 가능한 vCPU가 **30**개 이상인지 확인합니다. 사용 가능한 vCPU가 20개 이상인 경우에는 다음 연습부터 바로 진행하면 됩니다. 그렇지 않은 경우에는 이 연습의 다음 작업을 계속 진행합니다. 

#### <a name="task-2-request-vcpu-quota-increase"></a>작업 2: vCPU 할당량 늘리기 요청

1. Azure Portal에서 **구독**을 검색하여 선택하고 **구독** 블레이드에서 이 랩에 사용할 Azure 구독에 해당하는 항목을 선택합니다.
1. Azure Portal의 구독 블레이드 왼쪽 세로 메뉴에 있는 **설정** 섹션에서 **사용량 및 할당량**을 선택합니다. 

   **참고:** 할당량을 늘리기 위해 지원 티켓을 늘릴 필요가 없을 수도 있습니다.

   **참고:** 할당량 증가를 요청하려면 MFA(다단계 인증)를 사용하여 로그인해야 합니다. MFA를 사용하여 계정을 구성해야 하는 경우 [Azure Active Directory Multi-Factor Authentication 배포 계획](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-mfa-getstarted)을 참조하세요. 
   
1. **Azure Pass - 스폰서쉽 | 사용량 + 할당량** 블레이드에서 **지역**을 선택하고, 드롭다운 목록에서 이 랩에 사용할 Azure 지역 이름 옆에 있는 확인란을 선택합니다. 그리고 **적용**을 선택하고 **컴퓨팅** 항목이 **지역** 항목 왼쪽의 드롭다운 목록에 나타나는지 확인한 다음, 검색창에 **표준 BS**를 입력합니다. 
1. 결과 목록에서 **표준 BS 제품군 vCPU** 항목 옆의 확인란을 선택하고, 도구 모음에서 **할당량 증가 요청** 항목을 선택한 다음, 드롭다운 목록에서 **새 제한 입력**을 선택합니다.
1. **요청 할당량 증가** 창의 **새 제한** 열 텍스트 상자에 **30**을 입력한 다음, **제출**을 선택합니다.
1. 메시지가 표시되면 **할당량 증가 요청** 창에서 **다중 팩터리 인증으로 인증**을 선택하고 프롬프트에 따라 인증합니다.
1. 할당량 요청이 완료되도록 허용합니다.  잠시 후 **할당량 세부 정보** 블레이드에서 요청이 승인되고 할당량이 증가했음을 지정합니다. **할당량 세부 정보** 블레이드를 닫습니다.
1. 3~7단계를 반복하여 **표준 DSv3** VM 크기에 대한 할당량 제한을 **30**으로 설정합니다.

   >**참고**: Azure 지역 선택 및 현재 수요에 따라 지원 요청을 발생시켜야 할 수 있습니다. 지원 요청을 만드는 프로세스에 대한 지침은 [Azure 지원 요청 만들기])https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request) 를 참조하세요.

### <a name="exercise-1-deploy-an-active-directory-domain-services-ad-ds-domain"></a>연습 1: Active Directory Domain Services(AD DS) 도메인 배포

이 연습의 주요 작업은 다음과 같습니다.

1. Azure VM 배포 준비
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스트하는 Azure VM 배포
1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. Azure Bastion 배포

#### <a name="task-1-prepare-for-an-azure-vm-deployment"></a>작업 1: Azure VM 배포 준비

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal이 표시된 웹 브라우저에서 Azure AD 테넌트의 **개요** 블레이드로 이동한 후 왼쪽 세로 메뉴에 있는 **관리** 섹션에서 **속성**을 클릭합니다.
1. Azure AD 테넌트 **속성** 블레이드의 블레이드 맨 아래쪽에서 **보안 관리 기본값** 링크를 선택합니다.
1. **보안 기본값 사용** 블레이드에서 필요한 경우 **아니요**를 선택하고 **내 조직에서 조건부 액세스를 사용 중임** 체크박스를 선택한 후 **저장**을 선택합니다.
1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 


#### <a name="task-2-deploy-an-azure-vm-running-an-ad-ds-domain-controller-by-using-an-azure-resource-manager-quickstart-template"></a>작업 2: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Active Directory 도메인 컨트롤러를 호스트하는 Azure VM 배포

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 리소스 그룹을 만듭니다(`<Azure_region>` 자리 표시자를 이 랩에 사용할 Azure 지역 이름으로 바꿉니다(예: `eastus`)).

   ```powershell
   $location = '<Azure_region>'
   $resourceGroupName = 'az140-11-RG'
   New-AzResourceGroup -Location $location -Name $resourceGroupName
   ```

1. Azure Portal에서 **Cloud Shell** 창을 닫습니다.
1. 랩 컴퓨터의 동일한 웹 브라우저 창에서 다른 웹 브라우저 탭을 열고 [새 Windows VM 만들기 및 새 AD 포리스트, 도메인, DC 만들기](https://github.com/az140mp/azure-quickstart-templates/tree/master/application-workloads/active-directory/active-directory-new-domain)라는 빠른 시작 템플릿의 사용자 지정 버전을 탐색합니다. 
1. **새 Windows VM 만들기 및 새 AD 포리스트, 도메인 및 DC 만들기** 페이지에서 **Azure에 배포**를 선택합니다. 이렇게 하면 Azure Portal에 **새 AD 포리스트가 있는 Azure VM 만들기** 블레이드로 브라우저가 자동으로 리디렉션됩니다.
1. **새 AD 포리스트로 Azure VM 만들기** 블레이드에서 **매개 변수 편집**을 선택합니다.
1. **매개 변수 편집** 블레이드의 **열기** 대화 상자에서 **파일 로드**를 선택하고 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploydc11.parameters.json**을 선택한 후에 **열기**, **저장**을 차례로 선택합니다. 
1. **새 AD 포리스트를 사용하여 Azure VM 만들기** 블레이드에서 다음 설정을 지정합니다(나머지는 기존 값을 그대로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-11-RG**|
   |도메인 이름|**adatum.com**|

1. **새 AD 포리스트를 사용하여 Azure VM 만들기** 블레이드에서 **검토 + 만들기**, **만들기**를 차례로 선택합니다.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 15분 정도 걸릴 수 있습니다. 

#### <a name="task-3-deploy-an-azure-vm-running-windows-10-by-using-an-azure-resource-manager-quickstart-template"></a>작업 3: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

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
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 Windows 10을 실행하는 Azure VM을 배포합니다. 이 VM은 새로 만든 서브넷의 클라이언트로 사용됩니다.

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

#### <a name="task-4-deploy-azure-bastion"></a>작업 4: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

> **참고**: 브라우저에 팝업 기능이 사용하도록 설정되어 있는지 확인합니다.

1. Azure Portal을 표시하는 브라우저 창에서 다른 탭을 열고 브라우저 탭에서 Azure Portal로 이동합니다.
1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.
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

### <a name="exercise-2-integrate-an-ad-ds-forest-with-an-azure-ad-tenant"></a>연습 2: Azure AD 포리스트와 Azure AD 테넌트 통합
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure AD에 동기화할 AD DS 사용자 및 그룹 만들기
1. AD DS UPN 접미사 구성
1. Azure AD와의 동기화를 구성하는 데 사용할 Azure AD 사용자 만들기
1. Azure AD Connect 설치
1. 하이브리드 Azure AD 조인 구성

#### <a name="task-1-create-ad-ds-users-and-groups-that-will-be-synchronized-to-azure-ad"></a>작업 1: Azure AD에 동기화할 AD DS 사용자 및 그룹 만들기

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-dc-vm11**을 선택합니다.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**학생**|
   |암호|**Pa55w.rd1234**|

1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내에서 **Windows PowerShell ISE**를 관리자 권한으로 시작합니다.
1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 Internet Explorer 관리자용 보안 강화를 사용하지 않도록 설정합니다.

   ```powershell
   $adminRegEntry = 'HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}'
   Set-ItemProperty -Path $AdminRegEntry -Name 'IsInstalled' -Value 0
   Stop-Process -Name Explorer
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 AD DS 조직 구성 단위를 만듭니다. 이 조직 구성 단위에는 이 랩에서 사용하는 Azure AD 테넌트로의 동기화 범위 내 개체가 포함됩니다.

   ```powershell
   New-ADOrganizationalUnit 'ToSync' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 AD DS 조직 구성 단위를 만듭니다. 이 조직 구성 단위에는 Windows 10 도메인 조인 클라이언트 컴퓨터의 컴퓨터 개체가 포함됩니다.

   ```powershell
   New-ADOrganizationalUnit 'WVDClients' -path 'DC=adatum,DC=com' -ProtectedFromAccidentalDeletion $false
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 AD DS 사용자 계정을 만듭니다. 이 계정은 이 랩에서 사용하는 Azure AD 테넌트에 동기화됩니다(`<password>` 자리 표시자를 임의의 복잡한 암호로 교체).

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

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

   > **참고**: 이 스크립트는 권한이 없는 사용자 계정 9개(**aduser1** - **aduser9**), 그리고 **ADATUM\\도메인 관리자** 그룹 구성원인 권한이 있는 계정 1개(**wvdadmin1**)를 만듭니다.

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 AD DS 그룹 개체를 만듭니다. 이 개체는 이 랩에서 사용하는 Azure AD 테넌트에 동기화됩니다.

   ```powershell
   New-ADGroup -Name 'az140-wvd-pooled' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-remote-app' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-personal' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-users' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   New-ADGroup -Name 'az140-wvd-admins' -GroupScope 'Global' -GroupCategory Security -Path $ouPath
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 이전 단계에서 만든 그룹에 구성원을 추가합니다.

   ```powershell
   Get-ADGroup -Identity 'az140-wvd-pooled' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4'
   Get-ADGroup -Identity 'az140-wvd-remote-app' | Add-AdGroupMember -Members 'aduser1','aduser5','aduser6'
   Get-ADGroup -Identity 'az140-wvd-personal' | Add-AdGroupMember -Members 'aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-users' | Add-AdGroupMember -Members 'aduser1','aduser2','aduser3','aduser4','aduser5','aduser6','aduser7','aduser8','aduser9'
   Get-ADGroup -Identity 'az140-wvd-admins' | Add-AdGroupMember -Members 'wvdadmin1'
   ```

#### <a name="task-2-configure-ad-ds-upn-suffix"></a>작업 2: AD DS UPN 접미사 구성

1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 PowerShellGet 모듈의 최신 버전을 설치합니다(설치를 확인하라는 메시지가 표시되면 **예** 선택).

   ```powershell
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 Az PowerShell 모듈의 최신 버전을 설치합니다(설치를 확인하라는 메시지가 표시되면 **모두 예** 선택).

   ```powershell
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure 구독에 로그인합니다.

   ```powershell
   Connect-AzAccount
   ```

1. 메시지가 표시되면 이 랩에서 사용 중인 구독의 Owner 역할이 할당된 사용자 계정의 자격 증명을 입력합니다.
1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트의 Id 속성을 검색합니다.

   ```powershell
   $tenantId = (Get-AzContext).Tenant.Id
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure AD PowerSHell 모듈의 최신 버전을 설치하고 가져옵니다.

   ```powershell
   Install-Module -Name AzureAD -Force
   Import-Module -Name AzureAD
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure AD 테넌트에 인증합니다.

   ```powershell
   Connect-AzureAD -TenantId $tenantId
   ```

1. 메시지가 표시되면 이 작업 앞부분에서 사용한 것과 같은 자격 증명으로 로그인합니다. 
1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트의 기본 DNS 도메인 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트의 기본 DNS 도메인 이름을 AD DS 포리스트의 UPN 접미사 목록에 추가합니다.

   ```powershell
   Get-ADForest|Set-ADForest -UPNSuffixes @{add="$aadDomainName"}
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 Azure 구독과 연결된 Azure AD 테넌트의 기본 DNS 도메인 이름을 AD DS 도메인 내 모든 사용자의 UPN 접미사로 할당합니다.

   ```powershell
   $domainUsers = Get-ADUser -Filter {UserPrincipalName -like '*adatum.com'} -Properties userPrincipalName -ResultSetSize $null
   $domainUsers | foreach {$newUpn = $_.UserPrincipalName.Replace('adatum.com',$aadDomainName); $_ | Set-ADUser -UserPrincipalName $newUpn}
   ```

1. **관리자: Windows PowerShell ISE** 콘솔에서 다음 명령을 실행하여 **Student** 도메인 사용자에게 **adatum.com** UPN 접미사를 할당합니다.

   ```powershell
   $domainAdminUser = Get-ADUser -Filter {sAMAccountName -eq 'Student'} -Properties userPrincipalName
   $domainAdminUser | Set-ADUser -UserPrincipalName 'student@adatum.com'
   ```

#### <a name="task-3-create-an-azure-ad-user-that-will-be-used-to-configure-directory-synchronization"></a>작업 3: 디렉터리 동기화를 구성하는 데 사용할 Azure AD 사용자 만들기

1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 새 Azure AD 사용자를 만듭니다(`<password>` 자리 표시자를 임의의 복잡한 암호로 교체).

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $userName = 'aadsyncuser'
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName $userName -PasswordProfile $passwordProfile -MailNickName $userName -UserPrincipalName "$userName@$aadDomainName"
   ```

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 새로 만든 Azure AD 사용자에게 전역 관리자 역할을 할당합니다. 

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "$userName@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'} 
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **참고**: Azure AD PowerShell 모듈에서는 전역 관리자 역할의 명칭이 회사 관리자입니다.

1. **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 새로 만든 Azure AD 사용자의 사용자 계정 이름을 확인합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq '$userName'").UserPrincipalName
   ```

   > **참고**: 사용자 계정 이름을 적어 두세요. 나중에 이 연습에서 필요할 것입니다. 


#### <a name="task-4-install-azure-ad-connect"></a>작업 4: Azure AD Connect 설치

1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 TLS 1.2를 사용하도록 설정합니다.

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
   
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내에서 Internet Explorer를 시작하고 [비즈니스용 Microsoft Edge 다운로드 페이지](https://www.microsoft.com/en-us/edge/business/download)로 이동합니다.
1. [비즈니스용 Microsoft Edge 다운로드 페이지](https://www.microsoft.com/en-us/edge/business/download)에서 Microsoft Edge의 안정적인 최신 버전을 다운로드하여 설치 및 시작한 후 기본 설정을 사용하여 구성합니다.
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내에서 Microsoft Edge를 사용하여 [Azure Portal](https://portal.azure.com)로 이동합니다. 메시지가 표시되면 이 랩에서 사용 중인 구독의 Owner 역할이 할당된 사용자 계정의 Azure AD 자격 증명을 사용하여 로그인합니다.
1. Azure Portal에서 Azure Portal 페이지 상단의 **리소스, 서비스 및 문서 검색** 텍스트 상자를 사용하여 **Azure Active Directory** 블레이드를 검색한 후 해당 블레이드로 이동합니다. 그런 다음 Azure AD 테넌트 블레이드 허브 메뉴의 **관리** 섹션에서 **Azure AD Connect**를 선택합니다.
1. **Azure AD Connect** 블레이드에서 먼저 **동기화 연결** 링크를 선택한 다음 **Azure AD 연결 다운로드** 링크를 선택합니다. 그러면 새 브라우저 탭이 자동으로 열리고 **Microsoft Azure Active Directory Connect** 다운로드 페이지가 표시됩니다.
1. **Microsoft Azure Active Directory Connect** 다운로드 페이지에서 **다운로드**를 선택합니다.
1. **AzureADConnect.msi** 설치 관리자를 실행할지 아니면 저장할지 묻는 메시지가 표시되면 **실행**을 선택합니다. 그렇지 않으면 다운로드한 후 파일을 열어 **Microsoft Azure Active Directory 연결** 마법사를 시작합니다.
1. **Microsoft Azure Active Directory Connect** 마법사의 **Azure AD Connect 시작** 페이지에서 **라이선스 약관 및 개인 정보 보호 공지에 동의합니다**라는 체크박스를 선택하고 **계속**을 선택합니다.
1. **Microsoft Azure Active Directory Connect** 마법사의 **기본 설정** 페이지에서 **사용자 지정** 옵션을 선택합니다.
1. **필수 구성 요소 설치** 페이지에서 모든 선택적 구성 옵션의 선택을 취소하고 **설치**를 선택합니다.
1. **사용자 로그인** 페이지에서 **암호 해시 동기화**만 사용하도록 설정되었는지 확인하고 **다음**을 선택합니다.
1. **AD Azure에 연결** 페이지에서 이전 연습에서 만든 **aadsyncuser** 사용자 계정의 자격 증명을 사용하여 인증하고 **다음**을 선택합니다. 

   > **참고**: 이 연습의 앞부분에서 기록한 **aadsyncuser** 계정의 userPrincipalName 특성을 제공하고 이 랩의 앞부분에서 설정한 암호를 해당 암호로 지정합니다.

1. **디렉터리 연결** 페이지에서 **adatum.com** 포리스트 항목 오른쪽에 있는 **디렉터리 추가** 단추를 선택합니다.
1. **AD 포리스트 계정** 창에서 **새 AD 계정 만들기** 옵션이 선택되었는지 확인하고 다음 자격 증명을 지정한 후 **확인**을 선택합니다:

   |설정|값|
   |---|---|
   |사용자 이름|**ADATUM\Student**|
   |암호|**Pa55w.rd1234**|

1. **디렉터리 연결** 페이지에서 **adatum.com** 항목이 구성된 디렉터리로 표시되는지 확인하고 **다음**을 선택합니다
1. **Azure AD 로그인 구성** 페이지에서 **UPN 접미사가 확인된 도메인 이름과 일치하지 않으면 사용자가 해당 온-프레미스 자격 증명을 사용하여 Azure AD에 로그인할 수 없습니다**라는 경고를 확인하고, **모든 UPN 접미사를 확인된 도메인과 일치시키지 않고 계속합니다** 체크박스에 체크한 후 **다음**을 선택합니다.

   > **참고**: Azure AD 테넌트에 **adatum.com** AD DS의 UPN 접미사 중 하나와 일치하는 확인된 사용자 지정 DNS 도메인이 없으므로, 이 경고가 표시되는 것은 정상적인 현상입니다.

1. **도메인 및 OU 필터링** 페이지에서 **선택한 도메인 및 OU 동기화** 옵션을 선택하고 adatum.com 노드를 확장합니다. 그런 다음 모든 체크박스를 선택 취소하고 **ToSync** OU 옆에 있는 체크박스만 선택한 후 **다음**을 선택합니다.
1. **사용자를 고유하게 식별** 페이지에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **사용자 및 디바이스 필터링** 페이지에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **옵션 기능** 페이지에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **구성 준비 완료** 페이지에서 **구성이 완료되면 동기화 프로세스 시작** 체크박스가 선택되었는지 확인하고 **설치**를 선택합니다.

   > **참고**: 설치에는 약 2분이 소요됩니다.

1. **구성 완료** 페이지의 정보를 검토하고 **끝내기**를 선택하여 **Microsoft Azure Active Directory Connect** 창을 닫습니다.
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 Azure Portal가 표시된 Microsoft Edge 창에서 Adatum Lab Azure AD 테넌트의 **사용자 - 모든 사용자** 블레이드로 이동합니다.
1. **사용자 \| 모든 사용자** 블레이드에서 사용자 개체 목록에 이 랩 앞부분에서 만든 AD DS 사용자 계정 목록이 포함되어 있으며, **온-프레미스 동기화 사용 설정됨** 열에 **예** 항목이 표시되어 있음을 확인합니다.

   > **참고**: 몇 분 기다렸다가 브라우저 페이지를 새로 고쳐야 AD DS 사용자 계정이 표시될 수도 있습니다.
