---
lab:
  title: '랩: 이미지 템플릿을 사용하여 사용자 지정 세션 호스트 이미지 만들기'
  module: 'Module 1.5: Create and manage session host images'
---

# 랩 - 이미지 템플릿을 사용하여 사용자 지정 세션 호스트 이미지 만들기
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독
- 이 랩에서 사용할 Azure 구독에 대한 Owner 역할과, 해당 Azure 구독에 연결된 Microsoft Entra 테넌트에 디바이스를 조인하기에 충분한 사용 권한이 있는 Microsoft Entra 계정.

## 예상 소요 시간

90분 (약 45분은 이미지 빌드가 완료되는 데 걸리는 대기 시간임)

## 랩 시나리오

Azure Virtual Desktop 환경을 구현하려고 합니다. Azure Virtual Desktop 세션 호스트를 배포할 때 사용자 지정 가상 머신 이미지를 사용해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- 이미지 템플릿을 사용하여 Azure Virtual Desktop용 사용자 지정 세션 호스트 이미지 만들기

## 랩 파일

- None

## 지침

### 연습 1: 이미지 템플릿을 사용하여 사용자 지정 세션 호스트 이미지 만들기

이 연습의 주요 작업은 다음과 같습니다.

1. 필요한 리소스 공급자 등록
1. 사용자 할당 관리 ID 만들기
1. 사용자 지정 Azure RBAC(역할 기반 액세스 제어) 역할 만들기
1. 호스트 이미지 프로비저닝 관련 리소스에 대한 사용 권한 설정
1. Azure Compute Gallery 인스턴스 및 이미지 정의 만들기
1. 사용자 지정 이미지 템플릿 만들기
1. 사용자 지정 이미지 빌드
1. 사용자 지정 이미지를 사용하여 세션 호스트 배포

> **참고**: 사용자 지정 이미지 템플릿을 만들려면 다음을 비롯한 여러 필수 구성 요소를 충족해야 합니다.

- 모든 필 리소스 공급자를 등록합니다.
- 사용자 할당 관리 ID 만들기
- 사용자 지정 Azure RBAC(역할 기반 액세스 제어) 역할을 사용하여 사용자가 할당한 관리 ID에 필요한 권한을 부여합니다.
- Azure Compute Gallery를 사용하여 이미지를 배포하려면 이미지 정의와 함께 Azure Compute Gallery 인스턴스를 생성해야 합니다.

#### 작업 1: 필요한 리소스 공급자 등록

1. 필요한 경우 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal로 이동하여 이 랩에서 사용할 구독에서 Owner 역할을 가진 사용자 계정의 자격 증명을 제공하여 로그인합니다.

    > **참고**: 랩 세션 창의 오른쪽에 있는 리소스 탭에 나열된 `User1-` 계정의 자격 증명을 사용합니다.

1. Azure Portal의 Azure Cloud Shell에서 PowerShell 세션을 시작합니다.

    > **참고**: 메시지가 표시되면 **시작** 창의 **구독** 드롭다운 목록에서 이 랩에서 사용 중인 Azure 구독의 이름을 선택한 다음 **적용**을 선택합니다.

1. Azure Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 **Microsoft.DesktopVirtualization** 리소스 공급자를 등록합니다.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.DesktopVirtualization
    Register-AzResourceProvider -ProviderNamespace Microsoft.VirtualMachineImages
    Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
    Register-AzResourceProvider -ProviderNamespace Microsoft.Compute
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    Register-AzResourceProvider -ProviderNamespace Microsoft.KeyVault
    Register-AzResourceProvider -ProviderNamespace Microsoft.ContainerInstance
    ```

    > **참고**: 등록이 완료되기를 기다리지 마세요. 5분 정도 걸릴 수 있습니다.

1. Azure Cloud Shell 창을 닫습니다.

#### 작업 2: 사용자가 할당한 관리 ID 만들기

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **관리 ID**를 검색하여 선택합니다.
1. **관리 ID** 페이지에서 **+ 만들기**를 선택합니다.
1. **사용자가 할당한 관리 ID 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 지정한 후 **검토 + 만들기**를 선택합니다.

    > **참고**: **이름** 값을 설정할 때 랩 세션 창의 오른쪽에 있는 리소스 탭으로 전환하고 *User1-* 과 *@* 문자 사이의 문자열을 식별합니다. 이 문자열을 사용하여 *random* 자리 표시자를 바꿉니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|새 리소스 그룹의 이름 **az140-15a-RG**|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |속성|**az140**-*random*-**uami**|

1. **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    >**참고**: 사용자가 할당한 관리 ID의 프로비전이 완료되기를 기다릴 필요가 없습니다. 이 작업은 몇 밖에 걸리지 않습니다.

#### 작업 3: 사용자 지정 Azure RBAC(역할 기반 액세스 제어) 역할 만들기

>**참고**: 사용자 지정 Azure RBAC(역할 기반 액세스 제어) 역할은 이전 작업에서 만든, 사용자가 할당한 관리 ID에 적절한 사용 권한을 할당하는 데 사용됩니다.

1. 랩 컴퓨터에서 Azure Portal이 표시된 웹 브라우저를 열고 Azure Cloud Shell에서 PowerShell 세션을 시작합니다.

1. Azure Cloud Shell 창의 PowerShell 세션에서 다음 명령을 실행하여 이 랩에 사용되는 Azure 구독의 **Id** 속성 값을 식별하고 **$subscriptionId** 변수에 저장합니다.

    ```powershell
    $subscriptionId = (Get-AzSubscription).Id
    ```

1. 다음 명령을 실행하여 새 사용자 지정 역할의 역할 정의를 만들고(할당 가능한 범위 값 포함) **$jsonContent** 변수에 저장합니다(*random* 자리 표시자를 이전 작업에서 확인한 것과 동일한 문자열로 바꿔야 합니다).

    ```powershell
    $jsonContent = @"
    {
      "Name": "Desktop Virtualization Image Creator (random)",
      "IsCustom": true,
      "Description": "Create custom image templates for Azure Virtual Desktop images.",
      "Actions": [
        "Microsoft.Compute/galleries/read",
        "Microsoft.Compute/galleries/images/read",
        "Microsoft.Compute/galleries/images/versions/read",
        "Microsoft.Compute/galleries/images/versions/write",
        "Microsoft.Compute/images/write",
        "Microsoft.Compute/images/read",
        "Microsoft.Compute/images/delete"
      ],
      "NotActions": [],
      "DataActions": [],
      "NotDataActions": [],
      "AssignableScopes": [
        "/subscriptions/$subscriptionId",
        "/subscriptions/$subscriptionId/resourceGroups/az140-15b-RG"
      ]
    }
    "@
    ```

1. 다음 명령을 실행하여 **$jsonContent** 변수의 내용을 **CustomRole.json** 파일에 저장합니다.

    ```powershell
    $jsonContent | Out-File -FilePath 'CustomRole.json'
    ```

1. 다음 명령을 실행하여 사용자 지정 역할을 만듭니다.

    ```powershell
    New-AzRoleDefinition -InputFile ./CustomRole.json
    ```

1. Azure Cloud Shell 창을 닫습니다.

#### 작업 4: 호스트 이미지 프로비저닝 관련 리소스에 대한 사용 권한 설정

1. 랩 컴퓨터의 Azure Portal을 표시한 웹 브라우저에서 **리소스 그룹**을 검색하여 선택하고 **리소스 그룹** 페이지에서 **+ 만들기**를 선택합니다.
1. **리소스 그룹 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 지정한 후 **검토 + 만들기**를 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|새 리소스 그룹의 이름 **az140-15b-RG**|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|

1. **검토 + 만들기** 탭에서 **만들기**를 선택합니다.
1. **리소스 그룹** 페이지를 새로 고치고 리소스 그룹 목록에서 **az140-15b-RG**를 선택합니다.
1. **az140-15b-RG** 페이지의 세로 탐색 메뉴에서 **액세스 제어(IAM)** 를 선택합니다.
1. **az140-15b-RG\|액세스 제어(IAM)** 페이지에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **역할 할당 추가**를 선택합니다.
1. **역할 할당 추가** 페이지의 **역할** 탭에서 **작업 함수 역할** 탭이 선택되어 있는지 확인하고, 검색 텍스트 상자에 **데스크톱 가상화 이미지 작성자**(*임의*)를 입력하고, 결과 목록에서 **데스크톱 가상화 이미지 작성자**(*임의*)를 선택한 다음, **다음**을 선택합니다.

    >**참고**: *random* 자리표시자를 새 사용자 지정 RBAC 역할을 정의할 때 사용한 것과 동일한 문자열로 바꿔야 합니다.

1. **역할 할당 추가** 페이지의 **구성원** 탭에서 **관리 ID** 옵션을 선택하고, **+ 구성원 선택**을 클릭한 다음, **관리 ID 선택** 창의 **관리 ID** 드롭다운 목록에서 **사용자가 할당한 관리 ID**를 선택합니다. 그런 다음 사용자가 할당한 관리 ID 목록에서 **az140**-*random*-**uami**를 선택하고(여기서 *random*자리표시자가 새 사용자 지정 RBAC 역할을 정의할 때 사용한 것과 동일한 문자열을 나타냄), **선택**을 클릭합니다.
1. **역할 할당 추가** 페이지의 **구성원** 탭으로 돌아가서 **검토 + 할당**을 선택합니다.
1. **검토 + 할당** 탭에서 **검토 + 할당**을 선택합니다. 

#### 작업 5: Azure Compute Gallery 인스턴스 및 이미지 정의 만들기

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **Azure Compute Gallery**를 검색하여 선택하고 **Azure Compute Gallery** 페이지에서 **+ 만들기**를 선택합니다.
1. **Azure Compute Gallery** 만들기 페이지의 **기본 사항** 탭에서 다음 설정을 지정한 다음, **다음: 공유 방법**을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-15b-RG**|
    |속성|**az14015computegallery**|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|

1. **Azure Compute Gallery 만들기** 페이지의 **공유** 탭에서 기본 옵션인 **RBAC(역할 기반 액세스 제어)** 를 선택한 상태로 두고 **검토 + 만들기**를 선택합니다.
1. **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    >**참고**: 프로비저닝 프로세스가 완료될 때까지 기다립니다. 이는 1분 이내에 완료됩니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **Azure Compute Gallery**를 검색하여 선택하고, **Azure Compute Gallery** 페이지에서 **az14015computegallery**를 선택합니다. 
1. **az14015computegallery** 페이지에서 **+ 추가**를 선택하고 드롭다운 메뉴에서 **+ VM 이미지 정의**를 선택합니다. 
1. **VM 이미지 정의 만들기** 페이지의 **기본 사항** 탭에서 다음 설정을 지정하고(다른 설정은 기본값 유지) **다음: 버전**을 선택합니다.

    |설정|값|
    |---|---|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |VM 이미지 정의 이름|**az14015imagedefinition**|
    |OS 유형|**Windows**|
    |보안 유형|**신뢰할 수 있는 시작** 지원됨|
    |OS 상태|**일반화됨**|
    |게시자|**MicrosoftWindowsDesktop**|
    |제안|**Windows-11**|
    |SKU|**win11-23h2-avd-m365**|

    > **참고**: VM 세대는 자동으로 Gen2로 설정됩니다. 이는 Gen 1 가상 머신은 신뢰할 수 있음 및 기밀 보안 유형에서 지원되지 않기 때문입니다.

1. **VM 이미지 정의 만들기** 페이지의 **버전** 탭에서 설정을 변경하지 않고 **다음: 게시 옵션**을 선택합니다.

    > **참고**: 이 단계에서는 VM 이미지 버전을 만들면 안 됩니다. 이 작업은 Azure Virtual Desktop에서 수행됩니다.

1. **VM 이미지 정의 만들기** 페이지의 **게시 옵션** 탭에서 설정을 변경하지 않고 **검토 + 만들기**를 선택합니다.
1. **VM 이미지 정의 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 프로비저닝 프로세스가 완료될 때까지 기다립니다. 이 단계는 대개 1분 이내에 완료됩니다.

#### 작업 6: 사용자 지정 이미지 템플릿 만들기

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 페이지의 세로 탐색 메뉴에 있는 **관리** 섹션에서 **사용자 지정 이미지 템플릿**을 선택한 뒤 **Azure Virtual Desktop \| 사용자 지정 이미지 템플릿** 페이지에서 **+ 사용자 지정 이미지 템플릿 추가**를 선택합니다. 
1. **사용자 지정 이미지 템플릿** 페이지의 **기본 사항** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다.

    > **참고**: **관리 ID** 속성을 설정할 때 *random* 자리 표시자를 이 연습의 앞부분에서 확인한 것과 동일한 문자열로 바꿔야 합니다.

    |설정|값|
    |---|---|
    |템플릿 이름|**az140-15b-imagetemplate**|
    |기존 템플릿에서 가져오기|**No**|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-15b-RG**|
    |위치|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |관리 ID|**az140**-*random*-**uami**|

1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **원본 이미지** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |소스 형식|**플랫폼 이미지(마켓플레이스)**|
    |이미지 선택|**Windows 11 Enterprise 다중 세션, 버전 23H2 + Microsoft 365 Apps**|

1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **배포 대상** 탭에서 다음 설정을 지정하고(다른 설정은 기본값으로 유지) **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |Azure Compute Gallery|사용|
    |갤러리 이름|**az14015computegallery**|
    |갤러리 이미지 정의|**az14015imagedefinition**|
    |갤러리 이미지 버전|**1.0.0**|
    |출력 이름 실행|**az140-15-image-1.0.0**|
    |복제 지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |최신 항목에서 제외|**No**|
    |Storage 계정 유형|**Standard_LRS**|

    > **참고**: **복제 지역** 속성을 사용하여 다중 지역 빌드를 수용할 수 있습니다. **최신 항목에서 제외**를 **예**로 설정하면, VM 생성 시 **ImageReference** 요소의 버전으로 **최신**을 지정한 경우 이 이미지 버전을 사용하지 않습니다.

1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **빌드 속성** 탭에서 다음 설정을 지정하고(다른 설정은 기본값으로 유지) **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |빌드 시간 제한|**120**|
    |빌드 VM 크기|**Standard_DC2s_v3**|
    |OS 디스크 크기(GB)|**127**|
    |스테이징 그룹|**az140-15c-RG**|
    |VNet|설정되지 않은 상태로 |

    > **참고**: **스테이징 그룹**은 이미지를 빌드하고 로그를 저장하는 리소스를 스테이징하는 데 사용되는 리소스 그룹입니다. 이름을 입력하지 않으면 자동으로 생성됩니다. **VNet** 이름이 설정되지 않은 경우 빌드를 만드는 데 사용되는 VM의 공용 IP 주소와 함께 임시 이름이 만들어집니다.

    > **중요**: 지정한 빌드 VM 크기에 사용할 수 있는 vCPU 수가 충분한지 확인합니다. 그렇지 않은 경우 다른 크기를 선택하거나 할당량 증가를 요청합니다.

1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **사용자 지정** 탭에서 **+ 기본 제공 스크립트 추가**를 선택합니다. 
1. **기본 제공 스크립트 선택** 창에서 운영 체제별 스크립트, Azure Virtual Desktop 스크립트, MSIX 앱 연결 스크립트, 애플리케이션 스크립트, Windows 업데이트 관련 스크립트로 그룹화되어 있는 사용 가능한 옵션을 검토한 뒤 다음 항목을 선택합니다.

   - **표준 시간대 리디렉션**: 클라이언트가 세션 호스트의 세션 내에서 클라이언트의 표준 시간대를 사용하도록 허용
   - **저장 공간 센스 사용 안 함**: 저장 공간 센스가 여유 디스크 공간 부족 상태를 잘못 감지하여 세션 호스트에 부정적인 영향을 미치지 않도록 방지
   - **클라이언트 및 서버에서 화면 캡처 차단**을 통한 **화면 캡처 보호 사용**: 스크린샷 및 화면 공유에서 원격 콘텐츠를 차단하거나 숨김

1. **기본 제공 스크립트 선택** 창에서 **저장**을 선택합니다.

    > **참고**: 자신만의 스크립트를 추가할 수 있는 옵션이 있습니다. 예를 들어 [표준 시간대 리디렉션](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/TimezoneRedirection.ps1), [스토리지 센스 사용 안 함](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/DisableStorageSense.ps1) 또는 [화면 캡처 보호 사용](https://raw.githubusercontent.com/Azure/RDS-Templates/master/CustomImageTemplateScripts/CustomImageTemplateScripts_2024-03-27/ScreenCaptureProtection.ps1)과 같은 기본 제공 스크립트를 참조하는 것이 좋습니다.

1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **사용자 지정** 탭으로 돌아가서 **다음**을 선택합니다.
1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **태그** 탭에서 **다음**을 선택합니다.
1. **사용자 지정 이미지 템플릿 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 템플릿이 만들어질 때까지 기다립니다. 몇 분 정도 걸릴 수 있습니다. **Azure Virtual Desktop \| 사용자 지정 이미지 템플릿** 페이지를 새로 고쳐 템플릿 상태를 검토합니다.

#### 작업 7: 사용자 지정 이미지 빌드

> **참고**: 이 랩의 나머지 작업은 대기 시간이 상당히 길기 때문에 선택 사항입니다. 

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **Azure Virtual Desktop \| 사용자 지정 이미지 템플릿** 페이지로 이동해 **az140-15b-imagetemplate**을 선택합니다.
1. **az140-15b-imagetemplate** 페이지에서 **빌드 시작**을 선택합니다.

    > **참고**: 빌드가 만들어질 때까지 기다립니다. 빌드 프로세스를 완료하는 데 걸리는 실제 시간은 달라질 수 있지만 랩 지침에 제공된 설정을 사용하면 45분 이내에 왼료되어야 합니다. 몇 분마다 페이지를 새로 고치며 **az140-15b-imagetemplate** 페이지의 **Essentials** 섹션에 있는 **빌드 실행 상태** 값을 모니터링합니다. 

    > **참고**: 빌드 실행 상태는 어느 시점에 **실행 중 - 빌드 중**에서 **실행 중 - 배포 중**으로 변한 뒤 마지막으로 **성공**으로 변해야 합니다.

    > **참고**: 빌드가 완료될 때까지 기다리는 동안 스테이징 리소스 그룹 **az140-15c-RG**의 콘텐츠를 검토합니다. 여기에는 빌드 가상 머신, 가상 네트워크, 네트워크 보안 그룹, 키 자격 증명 모음, 스냅샷, 컨테이너 인스턴스, 스토리지 계정 등 빌드 리소스가 자동으로 프로비전됩니다. 

1. 랩 컴퓨터의 Azure Portal을 표시하는 웹 브라우저에서 **리소스 그룹**을 검색하여 선택하고 **리소스 그룹** 페이지에서 **az140-15c-RG**를 선택합니다.
1. **az140-15c-RG** 페이지의 **리소스** 섹션에서 자동 프로비된 리소스를 확인합니다.
1. **az140-15b-imagetemplate** 페이지로 돌아가 빌드 진행률을 모니터링합니다. 

    > **참고**: 다른 방법으로는 **활동 로그**를 사용하여 빌드 프로세스 완료를 추적할 수 있습니다. 집중 해야 하는 작업은 **VM 이미지 템플릿을 실행하여 출력을 생성**하는 것입니다. 이 작업의 상태는 어느 시점에 **수락됨**에서 **성공**으로 변해야 합니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **Azure Compute Gallery**를 검색하여 선택하고, **Azure Compute Gallery** 페이지에서 **az14015computegallery**를 선택합니다. 
1. **az14015computegallery**의 **정의** 탭에서 **az14015imagedefinition**을 선택합니다.
1. **az14015imagedefinition** 페이지의 **버전** 탭에서 **1.0.0(최신 버전)** 이미지에 대한 정보를 검토합니다.

#### 작업 8: 사용자 지정 이미지를 사용하여 세션 호스트 배포

> **참고**: 선택적으로, 생성한 사용자 지정 이미지를 사용하여 Azure Virtual Desktop 세션 호스트를 배포하는 초기 작업을 차례로 진행할 수도 있습니다. 

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저에서 **가상 네트워크**를 검색하여 선택하고 **가상 네트워크** 페이지에서 **만들기 +** 를 선택합니다.
1. **가상 네트워크 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음**을 선택합니다.

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|새 리소스 그룹의 이름 **az140-15d-RG**|
    |가상 네트워크 이름|**az140-vnet15d**|
    |지역|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|

1. **보안** 탭에서 기본 설정을 수락하고 **다음**을 선택합니다.
1. **IP 주소** 탭에서 다음을 설정을 지정합니다.

    |설정|값|
    |---|---|
    |IP 주소 공간|**10.30.0.0/16**|

1. **기본** 서브넷 항목 옆에 있는 편집(연필) 아이콘을 선택하고 **편집** 창에서 다음 설정을 지정하고(다른 설정을 기존 값으로 유지) **저장**을 선택합니다.

    |설정|값|
    |---|---|
    |속성|**hp1-Subnet**|
    |시작 주소|**10.30.1.0**|
    |프라이빗 서브넷 활성화(기본 아웃바운드 액세스 없음)|사용 안 함|

1. **IP 주소** 탭으로 돌아가서 **검토 + 만들기**를 선택한 다음 **만들기**를 선택합니다.

    > **참고**: 프로비저닝 프로세스가 완료될 때까지 기다립니다. 이 단계는 대개 1분 이내에 완료됩니다.

1. 랩 컴퓨터의 Azure Portal이 표시된 웹 브라우저 창에서 **Azure Virtual Desktop**을 검색하여 선택하고 **Azure Virtual Desktop** 페이지의 세로 탐색 메뉴의 **관리** 섹션에서 **호스트 풀**을 선택하고 **Azure Virtual Desktop \| 호스트 풀** 페이지에서 **+ 만들기**를 선택합니다. 
1. **호스트 풀 만들기** 페이지의 **기본** 탭에서 다음 설정을 지정하고 **다음: 세션 호스트 >** 을 선택합니다(다른 설정은 기본값 유지).

    |설정|값|
    |---|---|
    |구독|이 랩에서 사용 중인 Azure 구독의 이름|
    |Resource group|**az140-15d-RG**|
    |호스트 풀 이름|**az140-15-hp1**|
    |위치|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |유효성 검사 환경|**문제**|
    |기본 설정 앱 그룹 유형|**데스크톱**|
    |호스트 풀 유형|**풀링됨**|
    |세션 호스트 구성 만들기|**No**|
    |부하 분산 알고리즘|**너비 우선**|

    > **참고**: 폭 우선 부하 분산 알고리즘을 사용하는 경우 최대 세션 제한 매개 변수는 선택 사항입니다.

1. **호스트 풀 만들기** 페이지의 **세션 호스트** 탭에서 다음 설정을 지정합니다(다른 설정은 기본값으로 유지).

    > **참고**: **이름 접두사** 값을 설정할 때 랩 세션 창의 오른쪽에 있는 리소스 탭으로 전환하고 *User1*과 *@* 문자 사이의 문자열을 식별합니다. 이 문자열을 사용하여 *random* 자리 표시자를 바꿉니다.

    |설정|값|
    |---|---|
    |가상 컴퓨터 추가|**예**|
    |Resource group|**호스트 풀과 같은 그룹으로 기본 지정됨**|
    |이름 접두사|**sh0**_random_|
    |가상 머신 유형|**Azure 가상 머신**|
    |가상 머신 위치|Azure Virtual Desktop 환경을 배포하려는 Azure 지역의 이름|
    |가용성 옵션|**인프라 중복 필요 없음**|
    |보안 유형|**신뢰할 수 있는 시작 가상 머신**|

1. **호스트 풀 만들기** 페이지의 **가상 머신** 탭에서 **이미지** 드롭다운 목록 바로 아래에 있는 **모든 이미지 보기**를 선택합니다.
1. **이미지 선택** 페이지에서 **공유 이미지**를 선택하고 이미지 목록에서 **az14015imagedefinition**을 선택합니다. 
1. **호스트 풀 만들기** 페이지의 **가상 머신** 탭으로 돌아가서 다음 설정을 지정하고 **다음: 작업 영역 >** 을 선택합니다(다른 설정은 기본값 유지).

    |설정|값|
    |---|---|
    |가상 머신 크기|**표준 DC2s_v3**|
    |VM 수|**1**|
    |OS 디스크 유형|**표준 SSD**|
    |OS 디스크 크기|**기본 크기**|
    |부팅 진단|**관리형 스토리지 계정으로 활성화(권장)**|
    |가상 네트워크|az140-vnet15d|
    |서브넷|**hp1-Subnet**|
    |네트워크 보안 그룹|**기본**|
    |퍼블릭 인바운드 포트|**문제**|
    |조인할 디렉터리를 선택합니다.|**Microsoft Entra ID**|
    |Intune에 VM 등록|**No**|
    |사용자 이름|**Student**|
    |암호|기본 제공 관리자 계정의 암호로 사용될 충분히 복잡한 문자열|
    |암호 확인|이전에 지정한 것과 동일한 문자열|

    > **참고**: 암호는 길이가 12자 이상이어야 하며 소문자, 대문자, 숫자 및 특수 문자의 조합으로 구성됩니다. 자세한 내용은 [Azure VM을 만들 때의 암호 요구 사항](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm-)에 대한 정보를 참조하세요.

1. **호스트 풀 만들기** 페이지의 **작업 영역** 탭에서 다음 설정을 확인하고 **검토 + 만들기**를 선택합니다.

    |설정|값|
    |---|---|
    |데스크톱 앱 그룹 등록|**No**|

1. **호스트 풀 만들기** 페이지의 **검토 + 만들기** 탭에서 **만들기**를 선택합니다.

    > **참고**: 배포가 완료될 때까지 기다립니다. 10~15분 정도 걸릴 수 있습니다.
