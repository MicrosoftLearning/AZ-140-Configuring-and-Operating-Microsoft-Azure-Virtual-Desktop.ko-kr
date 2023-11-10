---
lab:
  title: '랩: Azure Virtual Desktop 배포 준비(Microsoft Entra DS)'
  module: 'Module 1: Plan an AVD Architecture'
---

# 랩 - Azure Virtual Desktop 배포 준비(Microsoft Entra DS)
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.

## 예상 소요 시간

150분

>**참고**: Microsoft Entra DS 프로비전에는 약 90분 대기 시간이 포함됩니다.

## 실습 시나리오

Azure Active Directory 도메인 Services(Microsoft Entra DS) 환경에서 Azure Virtual Desktop 배포를 준비해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Entra DS를 구현합니다기본
- Microsoft Entra DS do기본 환경 구성

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

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

1. Azure Portal의 Cloud Shell PowerShell 세션에서 **다음을 실행하여 등록되지 않은 경우 Microsoft.Compute** 리소스 공급자를 등록**합니다.**

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Azure Portal의 Cloud Shell PowerShell 세션에서 **다음을 실행하여 Microsoft.Compute** 리소스 공급자의 **등록 상태 확인합니다.**

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**참고**: 상태 등록됨으로 **나열되었는지** 확인합니다. 그렇지 않은 경우 몇 분 정도 기다렸다가 이 단계를 반복합니다.

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 Azure 지역 이름으로 PowerShell 변수를 만듭니다(자리 표시자를 이 랩에 사용하려는 Azure 지역 이름으로 바꿉 `<Azure_region>` 니다(예: 예: <a0 `eastus`/>).

   ```powershell
   $location = '<Azure_region>'
   ```

   > **참고**: Azure 지역의 이름을 식별하려면 Cloud Shell**의 **PowerShell 프롬프트에서 실행`(Get-AzLocation).Location`합니다.
   
1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 vCPU의 현재 사용량과 StandardDSv3Family** Azure VM에 대한 **해당 제한을 식별합니다.

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. 이전 단계에서 실행된 명령의 출력을 검토하고 대상 Azure 지역의 표준 DSv3 Azure VM 제품군**에 **20**개 이상의 **사용 가능한 vCPU가 있는지 확인합니다. 이미 해당되는 경우 다음 연습으로 직접 진행합니다. 그렇지 않으면 이 연습의 다음 작업을 진행합니다. 

#### 작업 2: vCPU 할당량 증가 요청

1. Azure Portal에서 구독**을 검색하여 선택하고 **구독 블레이드에서 **** 이 랩에 사용하려는 Azure 구독을 나타내는 항목을 선택합니다.
1. Azure Portal의 구독 블레이드 왼쪽에 있는 세로 **메뉴의 설정** 섹션에서 사용량 + 할당량을** 선택합니다**. 
1.  **Azure Pass - 스폰서쉽 | 사용량 + 할당량** 블레이드의 위쪽 검색 창에서 다음 드롭다운 화살표를 선택합니다.

   |**설정**|**값**|
   |---|---|
   |**Search**|**표준 DSv3**|
   |**모든 위치**|**모두 지운** 다음, *위치*를 확인합니다.|
   |**리소스 공급자** | **Microsoft.Compute** |
   
1. 반환 **된 표준 DSv3 제품군 vCPU** 항목에서 연필 아이콘인 **편집**을 선택합니다.
1. **할당량 세부 정보** 블레이드의 **새 제한** 열 텍스트 상자에 **30**을 입력한 다음, **저장 및 계속**을 선택합니다.
1. 할당량 요청이 완료되도록 허용합니다.  잠시 후 **할당량 세부 정보** 블레이드에서 요청이 승인되고 할당량이 증가했음을 지정합니다. **할당량 세부 정보** 블레이드를 닫습니다.

    >**참고**: Azure 지역 선택 및 현재 수요에 따라 지원 요청을 발생시키는 것이 필요할 수 있습니다. 지원 요청을 만드는 프로세스에 대한 지침은 Azure 지원 요청[ 만들기를 참조](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)하세요.

### 연습 1: AD DS(Azure Active Directory 도메인 Services)를 구현합니다기본

이 연습의 주요 작업은 다음과 같습니다.

1. Microsoft Entra DS 관리를 위한 Microsoft Entra 사용자 계정을 만들고 구성합니다기본
1. Azure Portal을 사용하여 Microsoft Entra DS 인스턴스 배포
1. Microsoft Entra DS 배포의 네트워크 및 ID 설정 구성

#### 작업 1: Microsoft Entra DS 관리를 위한 Microsoft Entra 사용자 계정을 만들고 구성합니다기본

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동한 다음, 이 랩에서 사용할 구독의 소유자 역할과 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할을 사용하여 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal을 표시하는 웹 브라우저에서 Microsoft Entra 테넌트 개요** 블레이드로 **이동하고 왼쪽**의 세로 메뉴에서 [관리**] 섹션에서 [속성]**을 클릭합니다**.
1. **Microsoft Entra 테넌트의 속성** 블레이드 블레이드 맨 아래에 있는 보안 관리 디올트** 링크를 선택합니다**.
1. 보안 기본값 사용 블레이드에서 **필요한 경우 아니요**를 선택하고 **내 조직에서 조건부 액세스** 검사 상자를 사용하고 있는지 선택하고 **저장**을 선택합니다**.**
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Microsoft Entra 테넌트에 로그인합니다.

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트에 대한 기본 DNS do기본 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Cloud Shell 창에서 다음을 실행하여 상승된 권한을 부여할 Microsoft Entra 사용자를 만듭니다(자리 표시자를 임의의 복잡한 암호로 바꿉 `<password>` 니다.)

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Cloud Shell 창에서 다음을 실행하여 새로 만든 Microsoft Entra 사용자 중 첫 번째 사용자에게 Global 관리istrator 역할을 할당합니다.

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **참고**: Azure AD PowerShell 모듈은 Global 관리istrator 역할을 회사 관리istrator로 나타냅니다.

1. Cloud Shell 창에서 다음을 실행하여 새로 만든 Microsoft Entra 사용자의 사용자 계정 이름을 식별합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **참고**: 사용자 계정 이름을 기록합니다. 나중에 이 연습에서 필요할 것입니다. 

1. Cloud Shell 창을 닫습니다.
1. Azure Portal 내에서 구독**을 검색하여 선택하고 **구독 블레이드에서 이 랩에서 **** 사용 중인 Azure 구독을 선택합니다. 
1. Azure 구독 속성이 표시된 블레이드에서 **액세스 제어(IAM)**, **추가**를 차례로 선택한 다음, **역할 할당 추가**를 선택합니다. 
1. **역할 할당 추가** 블레이드에서 **소유자**를 선택하고 **다음**을 클릭합니다.
1. **+멤버 선택** 하이퍼링크를 클릭합니다.
1. 구성원 선택 블레이드에서 aadadmin1** 항목을 선택한 **다음 선택** 단추를 클릭한 **다음 다음을 클릭합니다****.** ** 
1. **검토 + 할당** 블레이드에서 **검토 + 할당** 단추를 선택합니다.

   > **참고**: aadadmin1 계정을 사용하여 **나중에 랩에서 Windows 10** Azure VM에 가입된 Microsoft Entra DS의 Azure 구독 및 해당 Microsoft Entra 테넌트를 관리합니다. 


#### 작업 2: Azure Portal을 사용하여 Microsoft Entra DS 인스턴스 배포

1. 랩 컴퓨터의 Azure Portal에서 Microsoft Entra Do기본 Services를 검색하여 선택하고 **Microsoft Entra Do기본 Services** 블레이드에서 **+ 만들기**를 선택합니다**.** 그러면 Microsoft Entra Do기본 서비스 만들기 블레이드가** 열립니**다.
1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **기본** 사항 탭에서 다음 설정을 지정하고 다음**을 선택합니다**(다른 사용자는 기존 값으로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|새 **az140-11a-RG** 만들기 선택|
   |DNS 도메인 이름|**adatum.com**|
   |지역|AVD 배포를 호스트할 지역의 이름|
   |SKU|**표준**|

   > **참고**: 기술적으로는 필요하지 않지만 일반적으로 기존 Azure 또는 온-프레미스 DNS 이름 공간과 다른 Microsoft Entra DS기본 이름을 할당해야 합니다.

1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **네트워킹** 탭에서 가상 네트워크** 드롭다운 목록 옆에 있는 **새로** 만들기를 선택합니다**.
1. 가상 네트워크** 만들기 블레이드에서 **다음 설정을 할당하고 확인을** 선택합니다**.

   |설정|값|
   |---|---|
   |속성|**az140-aadds-vnet11a**|
   |주소 범위|**10.10.0.0/16**|
   |서브넷 이름|**aadds-Subnet**|
   |서브넷 이름|**10.10.0.0/24**|

1. **가상 네트워크** 만들기 블레이드의 **네트워킹** 탭으로 돌아가서 다음**을 선택합니다**(다른 사용자는 기존 값을 그대로 둡니다).
1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **관리사용** 탭에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **동기화** 탭에서 모두** 선택되었는지 확인하고 **다음**을 선택합니다**.
1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **보안 설정** 탭에서 기본 설정을 적용하고 다음**을 선택합니다**.
1. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **태그** 탭에서 기본 설정을 적용하고 다음을 선택합니다.
2. **Microsoft Entra Do기본 서비스** 만들기 블레이드의 **검토 + 만들기** 탭에서 만들기**를 선택합니다**. 
3. Microsoft Entra DS를 만든 후 변경할 수 없다는 설정에 대한 알림을 검토하고기본 확인을** 선택합니다**.

   >**참고**: Microsoft Entra DS를 프로비전한 후에 변경할 수 없는 설정에는 DNS 이름, Azure 구독, 리소스 그룹, 해당 도메인 네트워크 및 서브넷 호스팅하는 가상 네트워크 및 서브넷기본 컨트롤러 및 포리스트 유형이 포함됩니다기본.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 90분 정도 걸릴 수 있습니다. 

#### 작업 3: Microsoft Entra DS 배포의 네트워크 및 ID 설정 구성

1. 랩 컴퓨터의 Azure Portal에서 Microsoft Entra Do기본 Services**를 검색하여 선택하고 **Microsoft Entra Do기본 서비스** 블레이드에서 **adatum.com** 항목을 선택하여 **새로 프로비전된 Microsoft Entra DS 인스턴스로 이동합니다. 
1. **Microsoft Entra DS 인스턴스의 adatum.com** 블레이드에서 관리되는 작업에 대한 구성 문제로 시작하는 **경고를 클릭합니다기본 검색되었습니다**. 
1. **adatum.com | 구성 진단** 블레이드에서 실행을** 클릭합니다**.
1. **유효성** 검사 섹션에서 DNS 레코드** 창을 확장하고 **수정**을 클릭합니다**.
1. DNS 레코드 블레이드에서 **[수정 **]을 다시 클릭합니다**.**
1. Microsoft Entra DS 인스턴스**의 adatum.com** 블레이드로 돌아가 **필수 구성 단계** 섹션에서 Microsoft Entra DS 암호 해시 동기화에 대한 정보를 검토합니다. 

   > **참고**: Microsoft Entra DS에 액세스할 수 있어야 하는 기존 클라우드 전용 사용자는 컴퓨터와 리소스를 기본 암호를 변경하거나 재설정해야 합니다. 이는 이 랩의 앞부분 **에서 만든 aadadmin1** 계정에 적용됩니다.

1. 랩 컴퓨터의 Azure Portal에서 Cloud Shell 창에서 **PowerShell** 세션을 엽니다**.**
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Microsoft Entra **aadadmin1** 사용자 계정의 objectID 특성을 식별합니다.

   ```powershell
   Connect-AzureAD
   $objectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이전 단계에서 objectID를 확인한 **aadadmin1** 사용자 계정의 암호를 초기화합니다(`<password>` 자리 표시자를 임의의 복잡한 암호로 교체).

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $password = ConvertTo-SecureString '<password>' -AsPlainText -Force
   Set-AzureADUserPassword -ObjectId $objectId -Password $password -ForceChangePasswordNextLogin $false
   ```

   > **참고**: 실제 시나리오에서는 일반적으로 -ForceChangePasswordNextLogin**의 **값을 $true 설정합니다. 이 경우 랩 단계를 간소화하기 위해 $false** 선택**했습니다. 

1. 이전 두 단계를 반복하여 wvdaadmin1** 사용자 계정의 **암호를 다시 설정합니다.


### 연습 2: Microsoft Entra DS do기본 환경 구성
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. Azure Bastion 배포
1. Microsoft Entra DS의 기본 구성을 검토합니다기본
1. Microsoft Entra DS에 동기화될 AD DS 사용자 및 그룹 만들기

#### 작업 1: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

1. 랩 컴퓨터의 Azure Portal에서 Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 이전 작업에서 만든 az140-aadds-vnet11a**라는 가상 네트워크에 cl-Subnet이라는 **** 서브넷**을 추가합니다.

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'cl-Subnet' `
     -AddressPrefix 10.10.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. 랩 컴퓨터에서 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** 매개 변수 파일을 찾아 Visual Studio Code를 사용하여 파일을 엽니다.

1.  줄 21에서 domainPassword 매개 변수의 값을 찾습니다. **aadadmin1** 사용자 계정에 대해 이 랩의 앞부분에서 설정한 암호를 사용하도록 매개 변수 파일의 기존 암호를 업데이트한 다음, 파일을 **저장**합니다.

1. Azure Portal의 Cloud Shell 창 도구 모음에서 **파일 업로드/다운로드** 아이콘을 선택하고 드롭다운 메뉴에서 **업로드**를 선택합니다. 그런 다음, **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json** 및 **\\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json** 파일을 Cloud Shell 홈 디렉터리에 업로드합니다.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Azure Virtual Desktop 클라이언트 역할을 하는 Windows 10을 실행하는 Azure VM을 배포하고 Microsoft Entra DS에 조인합니다기본

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $location = (Get-AzResourceGroup -ResourceGroupName $resourceGroupName).Location
   New-AzResourceGroupDeployment `
     -ResourceGroupName $resourceGroupName `
     -Location $location `
     -Name az140lab0101vmDeployment `
     -TemplateFile $HOME/az140-11_azuredeploycl11a.json `
     -TemplateParameterFile $HOME/az140-11_azuredeploycl11a.parameters.json
   ```

   > **참고**: 배포는 10분 정도 걸릴 수 있습니다. 다음 작업을 진행하기 전에 배포가 완료되기를 기다립니다. 


#### 작업 2: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

> **참고**: 브라우저에 팝업 기능이 사용하도록 설정되어 있는지 확인합니다.

1. Azure Portal을 표시하는 브라우저 창에서 다른 탭을 열고 브라우저 탭에서 Azure Portal**[로 ](https://portal.azure.com)** 이동합니다.
1. Azure Portal에서 검색 텍스트 상자 오른쪽에 있는 도구 모음 아이콘을 선택하여 Cloud Shell** 창을 엽니다**.
1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 **AzureBastionSubnet**이라는 서브넷을 이 연습의 앞부분에서 만든 가상 네트워크 **az140-adds-vnet11**에 추가합니다.

   ```powershell
   $resourceGroupName = 'az140-11a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az140-aadds-vnet11a'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.10.254.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Cloud Shell 창을 닫습니다.
1. Azure Portal에서 **베스천**을 검색하여 선택하고 **베스천** 블레이드에서 **+ 만들기**를 선택합니다.
1. **베스천 만들기** 블레이드의 **기본 사항** 탭에서 다음 설정을 지정하고 **검토 + 만들기**를 선택합니다.

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|**az140-11a-RG**|
   |이름|**az140-11a-bastion**|
   |지역|이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역|
   |계층|**기본**|
   |가상 네트워크|**az140-aadds-vnet11a**|
   |서브넷|**AzureBastionSubnet(10.10.254.0/24)**|
   |공용 IP 주소|**새로 만들기**|
   |공용 IP 이름|**az140-aadds-vnet11a-ip**|

1. **베스천 만들기** 블레이드의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

   > **참고**: 이 연습의 다음 작업을 진행하기 전에 배포가 완료되기를 기다립니다. 배포는 5분 정도 걸릴 수 있습니다.


#### 작업 3: Microsoft Entra DS의 기본 구성을 검토합니다기본

> **참고**: 새로 Microsoft Entra DS에 가입된 컴퓨터에 로그인하려면 AAD DC 관리istrators** Microsoft Entra 그룹에 로그인**하려는 사용자 계정을 추가해야 합니다. 이 Microsoft Entra 그룹은 Microsoft Entra DS 인스턴스를 프로비전한 Azure 구독과 연결된 Microsoft Entra 테넌트에 자동으로 만들어집니다.

> **참고**: Microsoft Entra DS 인스턴스를 프로비전할 때 이 그룹을 기존 Microsoft Entra 사용자 계정으로 채울 수 있습니다.

1. 랩 컴퓨터의 Azure Portal에서 Cloud Shell 창에서 다음을 실행하여 aadadmin1** Microsoft Entra 사용자 계정을 **AAD DC 관리istrators** Microsoft Entra 그룹에 추가**합니다.

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell 창을 닫습니다.
1. 랩 컴퓨터의 Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-cl-vm11a** 항목을 선택합니다**. 그러면 az140-cl-vm11a 블레이드가 열립니 **다** .
1. **az140-cl-vm11a** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택한 다음, **az140-cl-vm11a**의 **베스천** 탭에서 다음 자격 증명을 제공하고 **연결**을 선택합니다.
1. 메시지가 표시되면 이 랩에서 이전에 식별한 주체 이름과 랩에서 이전에 만들 때 이 사용자 계정에 대해 설정한 암호를 사용하여 **aadadmin1** 사용자로 로그인합니다.
1. az140-cl-vm11a** Azure VM에 대한 **Bastion 내에서 Windows PowerShell ISE**를 관리istrator로 시작하고**, 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 Active Directory 및 DNS 관련 원격 서버 관리istration Tools를 설치합니다.

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **참고**: 다음 단계로 진행하기 전에 설치가 완료되기를 기다립니다. 2분 정도 걸릴 수 있습니다.

1. Bastion**에서 az140-cl-vm11a** Azure VM으로 이동한 후 **시작** 메뉴에서 Windows 관리istrative Tools** 폴더로 이동하여 **확장한 다음 도구 목록에서 Active Directory 사용자 및 컴퓨터** 시작**합니다. 
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 AADDC 컴퓨터** 및 AADDC 사용자** 조직 구성 단위를 **비롯한 **기본 계층 구조를 검토합니다. 전자는 az140-cl-vm11a** 컴퓨터 계정을 포함하고, 후자는 Microsoft Entra DS 인스턴스의 배포를 호스팅하는 Azure 구독과 연결된 Microsoft Entra 테넌트에서 동기화된 사용자 계정을 포함합니다**. **AADDC 사용자** 조직 구성 단위에는 그룹 멤버 자격과 함께 동일한 Microsoft Entra 테넌트에서 동기화된 AAD DC 관리istrators** 그룹도 포함**됩니다. 이 멤버 자격은 Microsoft Entra DS에서 직접 수정할 수 없으며기본 대신 Microsoft Entra DS 테넌트 내에서 관리해야 합니다. 모든 변경 내용은 Microsoft Entra DS에서 호스트되는 그룹의 복제본(replica) 자동으로 동기화됩니다기본. 

   **힌트:** Active Directory 사용자 및 컴퓨터** 할 일기본 관련 콘텐츠를 나열하지 않으면 **Active Directory 사용자 및 컴퓨터** 마우스 오른쪽 단추로 클릭하고 **변경 작업을 선택하고**기본** do기본 **Adatum**을 선택합니다.

   > **참고**: 현재 그룹에는 aadadmin1** 사용자 계정만 **포함됩니다.

1. Active Directory 사용자 및 컴퓨터 콘솔의 **AADDC 사용자** OU에서 **aadadmin1** 사용자 계정을 선택하고**, 속성 **** 대화 상자를 표시하고, 계정** 탭으로 **전환하고, 사용자 계정 이름 접미사가 기본 Microsoft Entra DNS와 일치하며기본 이름을 수정할 수 없습니다.** 
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 Do기본 Controllers** 조직 구성 단위의 콘텐츠를 검토하고 임의**로 생성된 이름을 가진 두 개의 do기본 컨트롤러의 컴퓨터 계정을 포함합니다. 

#### 작업 4: Microsoft Entra DS에 동기화될 AD DS 사용자 및 그룹 만들기

1. az140-cl-vm11a** Azure VM에 대한 Bastion **내에서 Microsoft Edge를 시작하고, Azure Portal[로 이동하고](https://portal.azure.com), aadadmin1** 사용자 계정의 사용자 계정 이름에 이 랩의 **앞부분에서 설정한 암호를 암호로 제공하여 로그인합니다.
1. Azure Portal에서 Cloud Shell**을 **엽니다.
1. Bash** 또는 PowerShell**을 **선택하라는 메시지가 표시되면 PowerShell**을 선택합니다****. 

   >**참고**: aadadmin1** 사용자 계정을 사용하여 **Cloud Shell**을 시작하는 **것은 이번이 처음이므로 Cloud Shell 홈 디렉터리를 구성해야 합니다. 탑재된 **스토리지** 메시지가 없는 경우 이 랩에서 사용 중인 구독을 선택하고 스토리지** 만들기를 선택합니다**. 

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 로그인하여 Microsoft Entra 테넌트에 인증합니다.

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트에 대한 기본 DNS do기본 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 예정된 랩에서 사용할 Microsoft Entra 사용자 계정을 만듭니다(자리 표시자를 임의의 복잡한 암호로 바꿉 `<password>` 니다.)

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   $aadUserNamePrefix = 'aaduser'
   $userCount = 1..9
   foreach ($counter in $userCount) {
     New-AzureADUser -AccountEnabled $true -DisplayName "$aadUserNamePrefix$counter" -PasswordProfile $passwordProfile -MailNickName "$aadUserNamePrefix$counter" -UserPrincipalName "$aadUserNamePrefix$counter@$aadDomainName"
   } 
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 az140-wvd-aadmins라는 **Microsoft Entra 그룹을 만들고 aadadmin1 및 **wvdaadmin1**** 사용자 계정에 추가 **** 합니다.

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell 창에서 이전 단계를 반복하여 예정된 랩에서 사용할 사용자에 대한 Microsoft Entra 그룹을 만들고 이전에 만든 Microsoft Entra 사용자 계정에 추가합니다.

   >**참고**: 가상 머신의 클립보드 크기가 제한되어 있으므로 나열된 모든 cmdlet이 올바르게 복사되지는 않습니다. 가상 머신에서 메모장을 열고 Lightning Bolt 컨트롤의 일부인 Type Text, Type Clipboard Text 구문을 사용하여 모든 cmdlet을 복사합니다. 모든 cmdlet이 메모장에 있는지 확인했으면 블록에서 잘라내어 Cloud Shell에 붙여넣고 실행합니다.

   ```powershell
   $az140wvdausers = New-AzureADGroup -Description 'az140-wvd-ausers' -DisplayName 'az140-wvd-ausers' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-ausers'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdausers.ObjectId -RefObjectId $userObjectId

   $az140wvdaremoteapp = New-AzureADGroup -Description "az140-wvd-aremote-app" -DisplayName "az140-wvd-aremote-app" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-aremote-app"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser5'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser6'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaremoteapp.ObjectId -RefObjectId $userObjectId

   $az140wvdapooled = New-AzureADGroup -Description "az140-wvd-apooled" -DisplayName "az140-wvd-apooled" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apooled"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser2'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser3'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser4'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapooled.ObjectId -RefObjectId $userObjectId

   $az140wvdapersonal = New-AzureADGroup -Description "az140-wvd-apersonal" -DisplayName "az140-wvd-apersonal" -MailEnabled $false -SecurityEnabled $true -MailNickName "az140-wvd-apersonal"
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser7'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser8'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aaduser9'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdapersonal.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell 창을 닫습니다.
1. az140-cl-vm11a** Azure VM에 대한 Bastion **내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Azure Active Directory** 블레이드를 검색하여 선택하고**, Microsoft Entra 테넌트 블레이드의 왼쪽에 있는 세로 메뉴 모음의 관리** 섹션에서 사용자를 선택하고 **, 사용자** \| 모두** 블레이드에서 **** 새 사용자 계정이 만들어졌는지 확인합니다.
1. Microsoft Entra 테넌트 블레이드로 돌아가서 왼쪽의 세로 메뉴 모음에서 **관리** 섹션에서 그룹을** 선택하고 **그룹 모든 그룹** 블레이드에서 **\| 새 그룹 계정이 만들어졌는지 확인합니다.
1. az140-cl-vm11a** Azure VM에 대한 Bastion **내에서 Active Directory 사용자 및 컴퓨터** 콘솔로 전환하고 **Active Directory 사용자 및 컴퓨터** 콘솔에서 **AADDC 사용자** OU로 이동하여 **동일한 사용자 및 그룹 계정이 포함되어 있는지 확인합니다.

   >**참고**: 콘솔 보기를 새로 고쳐야 할 수 있습니다.
