---
lab:
  title: '랩: Azure Virtual Desktop(Microsoft Entra DS) 배포 준비'
  module: 'Module 1: Plan an AVD Architecture'
---

# 랩 - Azure Virtual Desktop(Microsoft Entra DS) 배포 준비
# 학생용 랩 매뉴얼

## 랩 종속성

- Azure 구독
- Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할 및 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정.

## 예상 소요 시간

150분

>**참고**: Microsoft Entra DS 프로비전 시의 대기 시간은 약 90분입니다.

## 랩 시나리오

Azure Active Directory Domain Services(Microsoft Entra DS) 환경에서 Azure Virtual Desktop 배포를 준비해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Microsoft Entra DS 도메인 구현
- Microsoft Entra DS 도메인 환경 구성

## 랩 파일

-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.json
-  \\\\AZ-140\\AllFiles\\Labs\\01\\az140-11_azuredeploycl11a.parameters.json

## 지침

### 연습 0: vCPU 할당량 늘리기

이 연습의 주요 작업은 다음과 같습니다.

1. 현재 vCPU 사용량 파악
1. vCPU 할당량 늘리기 요청

#### 작업 1: 현재 vCPU 사용량 파악

1. 랩 컴퓨터에서 웹 브라우저를 시작하여 [Azure Portal](https://portal.azure.com)로 이동하고 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 검색 텍스트 상자 바로 오른쪽의 도구 모음 아이콘을 선택하여 **Cloud Shell** 창을 엽니다.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 **Microsoft.Compute** 리소스 공급자를 등록합니다(등록되지 않은 경우).

   ```powershell
   Register-AzResourceProvider -ProviderNamespace 'Microsoft.Compute'
   ```

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 **Microsoft.Compute** 리소스 공급자의 등록 상태를 확인합니다.

   ```powershell
   Get-AzResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Compute'}
   ```

   >**참고**: 상태가 **등록됨**로 나와 있는지 확인합니다. 그렇지 않은 경우 몇 분 기다렸다가 이 단계를 반복합니다.

1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 Azure 지역 이름으로 PowerShell 변수를 만듭니다(`<Azure_region>` 자리 표시자를 이 랩에 사용할 Azure 지역 이름으로 바꿈(예: `eastus`)).

   ```powershell
   $location = '<Azure_region>'
   ```

   > **참고**: Azure 지역의 이름을 확인하려면 **Cloud Shell**의 PowerShell 프롬프트에서 `(Get-AzLocation).Location`을 실행합니다.
   
1. Azure Portal의 **Cloud Shell** PowerShell 세션에서 다음을 실행하여 vCPU의 현재 사용량과 **StandardDSv3Family** Azure VM에 대한 해당 제한을 식별합니다.

   ```powershell
   Get-AzVMUsage -Location $location | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}
   ```

1. 이전 단계에서 실행한 명령의 출력을 검토하고 대상 Azure 지역의 Azure VM **표준 DSv3 제품군**에 사용 가능한 vCPU가 **20**개 이상 있는지 확인합니다. 사용 가능한 vCPU가 20개 이상인 경우에는 다음 연습부터 바로 진행하면 됩니다. 그렇지 않은 경우에는 이 연습의 다음 작업을 계속 진행합니다. 

#### 작업 2: vCPU 할당량 늘리기 요청

1. Azure Portal에서 **구독**을 검색하여 선택하고 **구독** 블레이드에서 이 랩에 사용할 Azure 구독에 해당하는 항목을 선택합니다.
1. Azure Portal의 구독 블레이드 왼쪽 세로 메뉴에 있는 **설정** 섹션에서 **사용량 및 할당량**을 선택합니다. 
1.  **사용량 + 할당량** 블레이드의 위쪽 검색 창에서 다음 드롭다운 화살표를 선택합니다.

   |**설정**|**값**|
   |---|---|
   |**Search**|**표준 DSv3**|
   |**모든 위치**|**모두 지운** 다음, *위치*를 확인합니다.|
   |**리소스 공급자** | **Microsoft.Compute** |
   
1. 반환된 **표준 DSv3 제품군 vCPU** 항목에서 연필 아이콘 **편집**을 선택합니다.
1. **할당량 세부 정보** 블레이드의 **새 제한** 열 텍스트 상자에 **30**을 입력한 다음, **저장 및 계속**을 선택합니다.
1. 할당량 요청이 완료되도록 허용합니다.  잠시 후 **할당량 세부 정보** 블레이드에서 요청이 승인되고 할당량이 증가했음을 지정합니다. **할당량 세부 정보** 블레이드를 닫습니다.

    >**참고**: Azure 지역 선택 및 현재 수요에 따라 지원 요청을 발생시켜야 할 수 있습니다. 지원 요청을 만드는 프로세스에 대한 지침은 [Azure 지원 요청 만들기](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)를 참조하세요.

### 연습 1: Azure Active Directory Domain Services(AD DS) 도메인 구현

이 연습의 주요 작업은 다음과 같습니다.

1. Microsoft Entra DS 도메인 관리를 위한 Microsoft Entra 사용자 계정 만들기 및 구성
1. Azure Portal을 사용하여 Microsoft Entra DS 인스턴스 배포
1. Microsoft Entra DS 배포의 네트워크 및 ID 설정 구성

#### 작업 1: Microsoft Entra DS 도메인 관리를 위한 Microsoft Entra 사용자 계정 만들기 및 구성

1. 랩 컴퓨터에서 웹 브라우저를 시작하고 [Azure Portal](https://portal.azure.com)로 이동합니다. 그런 다음 이 랩에서 사용할 구독의 Owner 역할, 그리고 Azure 구독과 연결된 Microsoft Entra 테넌트의 전역 관리자 역할이 할당된 사용자 계정의 자격 증명을 입력하여 로그인합니다.
1. Azure Portal이 표시되는 웹 브라우저에서 Microsoft Entra 테넌트의 **개요** 블레이드로 이동하고 왼쪽 세로 메뉴의 **관리** 섹션에서 **속성**을 클릭합니다.
1. Microsoft Entra 테넌트의 **속성** 블레이드 맨 아래에 있는 **보안 관리 기본값** 링크를 선택합니다.
1. **보안 기본값 사용** 블레이드에서 필요한 경우 **아니요**를 선택하고 **내 조직에서 조건부 액세스를 사용 중임** 체크박스를 선택한 후 **저장**을 선택합니다.
1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.
1. **Bash**와 **PowerShell** 중에서 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **Cloud Shell**을 처음 시작했는데 **탑재된 스토리지 없음**이라는 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창에서 다음을 실행하여 Microsoft Entra 테넌트에 로그인합니다.

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell 창에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트의 기본 DNS 도메인 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   $aadDomainName
   ```

1. Cloud Shell 창에서 다음을 실행하여 상승된 권한을 부여받을 Microsoft Entra 사용자를 만듭니다(`<password>` 자리 표시자를 임의의 복잡한 암호로 바꿈).

   > **참고**: 사용한 암호를 기억해야 합니다. 이 랩과 이후 랩에서 나중에 필요합니다.

   ```powershell
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = '<password>'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'aadadmin1' -PasswordProfile $passwordProfile -MailNickName 'aadadmin1' -UserPrincipalName "aadadmin1@$aadDomainName"
   New-AzureADUser -AccountEnabled $true -DisplayName 'wvdaadmin1' -PasswordProfile $passwordProfile -MailNickName 'wvdaadmin1' -UserPrincipalName "wvdaadmin1@$aadDomainName"
   ```

1. Cloud Shell 창에서 다음을 실행하여 새로 만들어진 Microsoft Entra 사용자 중 첫 번째 사용자에게 전역 관리자 역할을 할당합니다.

   ```powershell
   $aadUser = Get-AzureADUser -ObjectId "aadadmin1@$aadDomainName"
   $aadRole = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq 'Global administrator'}
   Add-AzureADDirectoryRoleMember -ObjectId $aadRole.ObjectId -RefObjectId $aadUser.ObjectId
   ```

   > **참고**: Azure AD PowerShell 모듈에서는 전역 관리자 역할의 명칭이 회사 관리자입니다.

1. Cloud Shell 창에서 다음을 실행하여 새로 만들어진 Microsoft Entra 사용자의 사용자 계정 이름을 식별합니다.

   ```powershell
   (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").UserPrincipalName
   ```

   > **참고**: 사용자 계정 이름을 적어 두세요. 나중에 이 연습에서 필요할 것입니다. 

1. Cloud Shell 창을 닫습니다.
1. Azure Portal 내에서 **구독**을 검색하여 선택하고 **구독** 블레이드에서 이 랩에 사용 중인 Azure 구독을 선택합니다. 
1. Azure 구독 속성이 표시된 블레이드에서 **액세스 제어(IAM)**, **추가**를 차례로 선택한 다음, **역할 할당 추가**를 선택합니다. 
1. **역할 할당 추가** 블레이드에서 **소유자**를 선택하고 **다음**을 클릭합니다.
1. **+멤버 선택** 하이퍼링크를 클릭합니다.
1. **구성원 선택** 블레이드에서 **aadadmin1** 항목을 선택한 다음, **선택** 단추를 클릭한 다음, **다음**을 클릭합니다.
1. **검토 + 할당** 블레이드에서 **검토 + 할당** 단추를 선택합니다.

   > **참고**: 랩 후반부에서는 **aadadmin1** 계정을 사용하여 Microsoft Entra DS에 조인된 Windows 10 Azure VM에서 Azure 구독과 해당 Microsoft Entra 테넌트를 관리하게 됩니다. 


#### 작업 2: Azure Portal을 사용하여 Microsoft Entra DS 인스턴스 배포

1. 랩 컴퓨터의 Azure Portal에서 **Microsoft Entra Domain Services**를 검색하여 선택하고, **Microsoft Entra Domain Services** 블레이드에서 **+ 만들기**를 선택합니다. 그러면 **Microsoft Entra Domain Services 만들기** 블레이드가 열립니다.
1. **Microsoft Entra Domain Services 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다(다른 항목은 기존 값 그대로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|새 **az140-11a-RG** 만들기 선택|
   |DNS 도메인 이름|**adatum.com**|
   |지역|AVD 배포를 호스트할 지역의 이름|
   |SKU|**표준**|

   > **참고**: 기술적 측면에서는 도메인 이름이 반드시 고유할 필요는 없지만, 일반적으로는 기존 Azure 또는 온-프레미스 DNS 이름 공간과 다른 Microsoft Entra DS 도메인 이름을 할당해야 합니다.

1. **Microsoft Entra Domain Services 만들기** 블레이드의 **네트워킹** 탭에서 **가상 네트워크** 드롭다운 목록 옆에 있는 **새로 만들기**를 선택합니다.
1. **가상 네트워크 만들기** 블레이드에서 다음 설정을 할당하고 **확인**을 선택합니다.

   |설정|값|
   |---|---|
   |속성|**az140-aadds-vnet11a**|
   |주소 범위|**10.10.0.0/16**|
   |서브넷 이름|**aadds-Subnet**|
   |주소 범위|**10.10.0.0/24**|

1. **가상 네트워크 만들기** 블레이드의 **네트워킹** 탭으로 돌아와 **다음**을 선택합니다(나머지는 기존 값을 그대로 유지).
1. **Microsoft Entra Domain Services 만들기** 블레이드의 **관리** 탭에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **Microsoft Entra Domain Services 만들기** 블레이드의 **동기화** 탭에서 **모두**가 선택되어 있는지 확인한 후 **다음**을 선택합니다.
1. **Microsoft Entra Domain Services 만들기** 블레이드의 **보안 설정** 탭에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **Microsoft Entra Domain Services 만들기** 블레이드의 **태그** 탭에서 기본 설정을 수락하고 다음을 선택합니다.
2. **Microsoft Entra Domain Services 만들기** 블레이드의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다. 
3. Microsoft Entra DS 도메인을 만든 후에는 변경할 수 없는 설정에 관한 알림을 검토하고 **확인**을 선택합니다.

   >**참고**: Microsoft Entra DS 도메인을 프로비전한 후에는 변경할 수 없는 설정으로는 도메인의 DNS 이름, Azure 구독, 리소스 그룹, 도메인 컨트롤러를 호스트하는 가상 네트워크와 서브넷, 포리스트 유형 등이 있습니다.

   > **참고**: 배포가 완료될 때까지 기다린 후 다음 연습을 진행하세요. 90분 정도 걸릴 수 있습니다. 

#### 작업 3: Microsoft Entra DS 배포의 네트워크 및 ID 설정 구성

1. 랩 컴퓨터의 Azure Portal에서 **Microsoft Entra Domain Services**를 검색 및 선택하고, **Microsoft Entra Domain Services** 블레이드에서 **adatum.com** 항목을 선택하여 새로 프로비전된 Microsoft Entra DS 인스턴스로 이동합니다. 
1. Microsoft Entra DS 인스턴스의 **adatum.com** 블레이드에서 **관리되는 도메인에서 구성 문제가 감지되었음**으로 시작하는 경고를 클릭합니다. 
1. **adatum.com | 구성 진단** 블레이드에서 **실행**을 클릭합니다.
1. **유효성 검사** 섹션에서 **DNS 레코드** 창을 확장하고 **수정**을 클릭합니다.
1. **DNS 레코드** 블레이드에서 **수정**을 다시 클릭합니다.
1. Microsoft Entra DS 인스턴스의 **adatum.com** 블레이드로 다시 이동하여 **필수 구성 단계** 섹션에서 Microsoft Entra DS 암호 해시 동기화에 관한 정보를 검토합니다. 

   > **참고**: Microsoft Entra DS 도메인 컴퓨터 및 해당 리소스에 액세스할 수 있어야 하는 기존 클라우드 전용 사용자는 암호를 변경하거나 재설정해야 합니다. 이 랩의 앞부분에서 만든 **aadadmin1** 계정도 마찬가지입니다.

1. 랩 컴퓨터에 표시된 Azure Portal의 **Cloud Shell** 창에서 **PowerShell **세션을 엽니다.
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

   > **참고**: 실제 시나리오에서는 대개 **-ForceChangePasswordNextLogin**의 값을 $true로 설정합니다. 여기서는 랩 단계를 간편하게 실행하기 위해 값으로 **$false**를 선택했습니다. 

1. 이전 두 단계를 반복하여 **wvdaadmin1** 사용자 계정의 암호를 재설정합니다.


### 연습 2: Microsoft Entra DS 도메인 환경 구성
  
이 연습의 주요 작업은 다음과 같습니다.

1. Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포
1. Azure Bastion 배포
1. Microsoft Entra DS 도메인의 기본 구성 검토
1. Microsoft Entra DS와 동기화할 AD DS 사용자 및 그룹 만들기

#### 작업 1: Azure Resource Manager 빠른 시작 템플릿을 사용하여 Windows 10을 실행하는 Azure VM 배포

1. 랩 컴퓨터에 표시된 Azure Portal 내 Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이전 작업에서 만든 **az140-aadds-vnet11a** 가상 네트워크에 서브넷 **cl-Subnet**을 추가합니다.

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
1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 Windows 10을 실행하는 Azure VM을 배포합니다. 이 VM은 Azure Virtual Desktop 클라이언트로 사용되며, Microsoft Entra DS 도메인에 Azure Virtual Desktop 클라이언트를 연결합니다.

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

   > **참고**: 배포는 10분 정도 걸릴 수 있습니다. 다음 작업을 진행하기 전에 배포가 완료될 때까지 기다립니다. 


#### 작업 2: Azure Bastion 배포 

> **참고**: Azure Bastion은 이 연습의 이전 작업에서 배포한 퍼블릭 엔드포인트 없이 Azure VM에 연결할 수 있으며 운영 체제 수준 자격 증명을 무차별 악용하지 못하도록 보호합니다.

> **참고**: 브라우저에 팝업 기능이 사용하도록 설정되어 있는지 확인합니다.

1. Azure Portal을 표시하는 브라우저 창에서 다른 탭을 열고 브라우저 탭에서 [**Azure Portal**](https://portal.azure.com)로 이동합니다.
1. Azure Portal에서 검색 텍스트 상자의 오른쪽에 있는 도구 모음 아이콘을 직접 선택하여 **Cloud Shell** 창을 엽니다.
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
   |속성|**az140-11a-bastion**|
   |지역|이 연습의 이전 작업에서 리소스를 배포한 동일한 Azure 지역|
   |계층|**기본**|
   |가상 네트워크|**az140-aadds-vnet11a**|
   |서브넷|**AzureBastionSubnet(10.10.254.0/24)**|
   |공용 IP 주소|**새로 만들기**|
   |공용 IP 이름|**az140-aadds-vnet11a-ip**|

1. **베스천 만들기** 블레이드의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

   > **참고**: 이 연습의 다음 작업을 진행하기 전에 배포가 완료될 때까지 기다립니다. 배포는 5분 정도 걸릴 수 있습니다.


#### 작업 3: Microsoft Entra DS 도메인의 기본 구성 검토

> **참고**: 새로 Microsoft Entra DS에 조인한 컴퓨터에 로그인하려면 먼저 로그인하려는 사용자 계정을 **AAD DC Administrators** Microsoft Entra 그룹에 추가해야 합니다. 이 Microsoft Entra 그룹은 Microsoft Entra DS 인스턴스를 프로비전한 Azure 구독과 연결된 Microsoft Entra 테넌트에 자동으로 만들어집니다.

> **참고**: Microsoft Entra DS 인스턴스를 프로비전할 때 이 그룹에 기존 Microsoft Entra 사용자 계정을 추가할 수 있습니다.

1. 랩 컴퓨터의 Azure Portal Cloud Shell 창에서 다음을 실행하여 **aadadmin1** Microsoft Entra 사용자 계정을 **AAD DC Administrators** Microsoft Entra 그룹에 추가합니다.

   ```powershell
   Connect-AzureAD
   $groupObjectId = (Get-AzureADGroup -Filter "DisplayName eq 'AAD DC Administrators'").ObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $groupObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell 창을 닫습니다.
1. 랩 컴퓨터에 표시된 Azure Portal에서 **가상 머신**을 검색하여 선택하고 **가상 머신** 블레이드에서 **az140-cl-vm11a** 항목을 선택합니다. 그러면 **az140-cl-vm11a** 블레이드가 열립니다.
1. **az140-cl-vm11a** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택한 다음, **az140-cl-vm11a**의 **베스천** 탭에서 다음 자격 증명을 제공하고 **연결**을 선택합니다.
1. 메시지가 표시되면 이 랩에서 이전에 식별한 주체 이름과 랩에서 이전에 만들 때 이 사용자 계정에 대해 설정한 암호를 사용하여 **aadadmin1** 사용자로 로그인합니다.
1. **az140-cl-vm11a** Azure VM에 연결된 Bastion 내에서 관리자로 **Windows PowerShell ISE**를 시작하고 **관리자: Windows PowerShell ISE** 스크립트 창에서 다음 명령을 실행하여 Active Directory 및 DNS 관련 원격 서버 관리 도구를 설치합니다.

   ```powershell
   Add-WindowsCapability -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.Dns.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0 -Online
   Add-WindowsCapability -Name Rsat.ServerManager.Tools~~~~0.0.1.0 -Online
   ```

   > **참고**: 설치가 완료될 때까지 기다렸다가 다음 단계를 진행합니다. 2분 정도 걸릴 수 있습니다.

1. **az140-cl-vm11a** Azure VM에 연결된 원격 데스크톱 내의 **시작** 메뉴에서 **Windows 관리 도구** 폴더로 이동하여 해당 폴더를 확장하고 도구 목록에서 **Active Directory 사용자 및 컴퓨터**를 선택합니다. 
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 **AADDC Computers** 및 **AADDC Users** 조직 구성 단위를 포함한 기본 계층 구조를 검토합니다. AADDC Computers 조직 구성 단위에는 **az140-cl-vm11a** 컴퓨터 계정이 포함되어 있습니다. 그리고 AADDC Users 조직 구성 단위에는 Microsoft Entra DS 인스턴스 배포를 호스트하는 Azure 구독과 연결된 Microsoft Entra 테넌트에서 동기화되는 사용자 계정이 포함되어 있습니다. **AADDC Users** 조직 구성 단위에는 그룹 멤버 자격과 함께 동일한 Microsoft Entra 테넌트에서 동기화된 **AAD DC Administrators** 그룹도 포함됩니다. 이 멤버 자격은 Microsoft Entra DS 도메인 내에서 직접 수정할 수 없으며 대신 Microsoft Entra DS 테넌트 내에서 관리해야 합니다. 모든 변경 내용은 Microsoft Entra DS 도메인에서 호스트되는 그룹의 복제본과 자동으로 동기화됩니다. 

   **힌트:** **Active Directory 사용자 및 컴퓨터**에 도메인 관련 콘텐츠가 나열되지 않으면 **Active Directory 사용자 및 컴퓨터**를 마우스 오른쪽 단추로 클릭하고 **도메인 변경**을 선택한 다음, **Adatum** 도메인을 선택합니다.

   > **참고**: 현재 이 그룹에는 **aadadmin1** 사용자 계정만 포함되어 있습니다.

1. **Active Directory 사용자 및 컴퓨터** 콘솔의 **AADDC Users** OU에서 **aadadmin1** 사용자 계정을 선택하여 해당 **속성** 대화 상자를 표시합니다. 그런 다음 **계정** 탭으로 전환하여 사용자 계정 이름 접미사가 기본 Microsoft Entra DNS 도메인 이름과 일치하며 수정 불가능한 상태임을 확인합니다. 
1. **Active Directory 사용자 및 컴퓨터** 콘솔에서 **Domain Controllers** 조직 구성 단위의 내용을 검토하여 임의로 생성된 이름이 지정되어 있는 도메인 컨트롤러 2개의 컴퓨터 계정이 포함되어 있음을 확인합니다. 

#### 작업 4: Microsoft Entra DS와 동기화할 AD DS 사용자 및 그룹 만들기

1. **az140-cl-vm11a** Azure VM에 연결된 Bastion 내에서 Microsoft Edge를 시작하고 [Azure Portal](https://portal.azure.com)로 이동한 다음, 이 랩의 앞부분에서 암호로 설정한 암호를 사용하여 **aadadmin1** 사용자 계정의 사용자 계정 이름을 제공하여 로그인합니다.
1. Azure Portal에서 **Cloud Shell**을 엽니다.
1. **Bash** 또는 **PowerShell**을 선택하라는 메시지가 표시되면 **PowerShell**을 선택합니다. 

   >**참고**: **aadadmin1** 사용자 계정을 사용하여 **Cloud Shell**을 처음 시작하는 경우 Cloud Shell 홈 디렉터리를 구성해야 합니다. **탑재된 스토리지가 없음** 메시지가 표시되면 이 랩에서 사용하는 구독을 선택하고 **스토리지 만들기**를 선택합니다. 

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Microsoft Entra 테넌트에 로그인한 후 인증합니다.

   ```powershell
   Connect-AzureAD
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 Azure 구독과 연결된 Microsoft Entra 테넌트의 기본 DNS 도메인 이름을 검색합니다.

   ```powershell
   $aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이후 랩에서 사용할 Microsoft Entra 사용자 계정을 만듭니다(`<password>` 자리 표시자를 임의의 복잡한 암호로 교체).

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

1. Cloud Shell 창의 PowerShell 세션에서 다음을 실행하여 **az140-wvd-aadmins**라는 Microsoft Entra 그룹을 만들고 여기에 **aadadmin1** 및 **wvdaadmin1** 사용자 계정을 추가합니다.

   ```powershell
   $az140wvdaadmins = New-AzureADGroup -Description 'az140-wvd-aadmins' -DisplayName 'az140-wvd-aadmins' -MailEnabled $false -SecurityEnabled $true -MailNickName 'az140-wvd-aadmins'
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'aadadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   $userObjectId = (Get-AzureADUser -Filter "MailNickName eq 'wvdaadmin1'").ObjectId
   Add-AzureADGroupMember -ObjectId $az140wvdaadmins.ObjectId -RefObjectId $userObjectId
   ```

1. Cloud Shell 창에서 이전 단계를 반복하여 예정된 랩에서 사용할 사용자에 대한 Microsoft Entra 그룹을 만들고 이전에 만든 Microsoft Entra 사용자 계정에 추가합니다.

   >**참고**: 참고: 가상 머신의 클립보드 크기가 제한되어 있으므로 나열된 모든 cmdlet이 올바르게 복사되지는 않습니다. 가상 머신에서 메모장을 열고 Lightning Bolt 컨트롤의 일부인 Type Text, Type Clipboard Text 구문을 사용하여 모든 cmdlet을 복사합니다. 모든 cmdlet이 메모장에 있는지 확인했으면 블록에서 잘라내어 Cloud Shell에 붙여넣고 실행합니다.

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
1. **az140-cl-vm11a** Azure VM에 연결된 Bastion 내의 Azure Portal이 표시된 Microsoft Edge 창에서 **Azure Active Directory** 블레이드를 검색하여 선택합니다. 그런 다음, Microsoft Entra 테넌트 블레이드 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **사용자**를 선택합니다. 그런 다음, **사용자 \| 모든 사용자** 블레이드에서 새 사용자 계정이 만들어졌는지 확인합니다.
1. Microsoft Entra 테넌트 블레이드로 돌아가서 왼쪽의 세로 메뉴 모음에 있는 **관리** 섹션에서 **그룹**을 선택한 다음, **그룹 \| 모든 그룹** 블레이드에서 새 그룹 계정이 만들어졌는지 확인합니다.
1. **az140-cl-vm11a** Azure VM에 연결된 원격 데스크톱 내에서 **Active Directory 사용자 및 컴퓨터** 콘솔로 전환합니다. 그런 다음 **Active Directory 사용자 및 컴퓨터** 콘솔에서 **AADDC Users** OU로 이동하여 동일한 사용자 및 그룹 계정이 포함되어 있는지 확인합니다.

   >**참고**: 콘솔 보기를 새로 고쳐야 할 수도 있습니다.
