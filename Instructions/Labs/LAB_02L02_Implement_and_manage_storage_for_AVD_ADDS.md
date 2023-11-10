---
lab:
  title: '랩: AVD(AD DS)에 대한 스토리지 구현 및 관리'
  module: 'Module 2: Implement an AVD Infrastructure'
---

# 랩 - AVD(AD DS)에 대한 스토리지 구현 및 관리
# 학생용 랩 매뉴얼

## 랩 종속성

- 이 랩에서 사용할 Azure 구독입니다.
- 이 랩에서 사용할 Azure 구독의 소유자 또는 기여자 역할이 있는 Microsoft 계정 또는 Microsoft Entra 계정 및 해당 Azure 구독과 연결된 Microsoft Entra 테넌트의 Global 관리istrator 역할.
- 완료된 **Azure Virtual Desktop의 배포 준비(AD DS)** 랩

## 예상 소요 시간

30분

## 실습 시나리오

Microsoft Entra DS 환경에서 Azure Virtual Desktop 배포에 대한 스토리지를 구현하고 관리해야 합니다.

## 목표
  
이 랩을 완료하면 다음을 수행할 수 있습니다.

- Azure Virtual Desktop용 프로필 컨테이너를 저장하도록 Azure Files 구성

## 랩 파일

- 없음

## 지침

### 연습 1: Azure Virtual Desktop용 프로필 컨테이너를 저장하도록 Azure Files 구성

이 연습의 주요 작업은 다음과 같습니다.

1. Azure Storage 계정 만들기
1. Azure Files 공유 만들기
1. Azure Storage 계정에 대한 AD DS 인증 사용 
1. Azure Files RBAC 기반 권한 구성
1. Azure Files 파일 시스템 권한 구성

#### 작업 1: Azure Storage 계정 만들기

1. 랩 컴퓨터에서 웹 브라우저를 시작하고, Azure Portal[로 ](https://portal.azure.com)이동하고, 이 랩에서 사용할 구독에서 소유자 역할이 있는 사용자 계정의 자격 증명을 제공하여 로그인합니다.
1. Azure Portal에서 가상 머신을 검색하여 선택하고 **가상 머신**** 블레이드에서 **az140-dc-vm11**을 선택합니다**.
1. **az140-dc-vm11** 블레이드에서 **연결**을 선택하고 드롭다운 메뉴에서 **베스천**을 선택합니다. **az140-dc-vm11 \| 연결** 블레이드의 **베스천** 탭에서 **베스천 사용**을 선택합니다.
1. 메시지가 표시되면 다음 자격 증명을 제공하고 **연결**을 선택합니다.

   |설정|값|
   |---|---|
   |사용자 이름|**Student@adatum.com**|
   |암호|**Pa55w.rd1234**|

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Microsoft Edge를 시작하고 Azure Portal[로 이동합니다](https://portal.azure.com).** 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할이 있는 사용자 계정의 Microsoft Entra 자격 증명을 사용하여 로그인합니다.
1. **az140-dc-vm11**에 연결된 원격 데스크톱 세션 내의 Azure Portal이 표시된 Microsoft Edge 창에서 **스토리지 계정**을 검색하여 선택한 후 **스토리지 계정** 블레이드에서 **+ 만들기**를 선택합니다.
1. **스토리지 계정 만들기** 블레이드의 **기본** 탭에서 다음 설정을 지정합니다(나머지는 기본값을 그대로 유지).

   |설정|값|
   |---|---|
   |구독|이 랩에서 사용 중인 Azure 구독의 이름|
   |Resource group|새 리소스 그룹 **az140-22-RG**의 이름|
   |스토리지 계정 이름|문자로 시작하는 소문자와 숫자로 구성된 3에서 15 사이의 전역적으로 고유한 이름|
   |지역|Azure Virtual Desktop 랩 환경을 호스팅하는 Azure 지역의 이름|
   |성능|**Standard**|
   |중복|**GRS(지역 중복 스토리지)**|
   |지역 가용성이 없는 경우 데이터에 대한 읽기 액세스 권한 부여|사용|

   >**참고**: 스토리지 계정 이름의 길이가 15자를 초과하지 않는지 확인합니다. 이 이름은 스토리지 계정을 포함하는 Azure 구독과 연결된 Microsoft Entra 테넌트에 통합된 AD DS(Active Directory 도메인 Services) do기본에서 컴퓨터 계정을 만드는 데 사용됩니다. 이렇게 하면 이 스토리지 계정에서 호스트되는 파일 공유에 액세스할 때 AD DS 기반 인증이 허용됩니다.

1. **스토리지 계정 만들기** 블레이드의 **기본 사항** 탭에서 **검토 + 만들기**를 선택하고, 유효성 검사 프로세스가 완료될 때까지 기다린 다음, **만들기**를 선택합니다.

   >**참고**: 스토리지 계정이 만들어질 때까지 기다립니다. 이 작업은 2분 정도 걸립니다.

#### 작업 2: Azure Files 공유 만들기

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창에서 Storage 계정 블레이드로 **다시 이동하여 새로 만든 스토리지 계정을** 나타내는 항목을** 선택합니다.
1. 스토리지 계정 블레이드의 **데이터 스토리지** 섹션에서 파일 공유**를 선택한 **다음+ 파일 공유**를 선택합니다**.
1. 새 파일 공유 블레이드에서 **다음 설정을 지정하고 만들기**를 선택합니다**(다른 설정은 기본값으로 유지**).

   |설정|값|
   |---|---|
   |속성|**az140-22-profiles**|
   |계층|**트랜잭션 최적화됨**|

#### 작업 3: Azure Storage 계정에 AD DS 인증 사용 

1. **az140-dc-vm11**에 연결된 원격 데스크톱 내의 Microsoft Edge 창에서 다른 탭을 열고 [Azure Files 샘플 GitHub 리포지토리](https://github.com/Azure-Samples/azure-files-samples/releases)로 이동합니다. 그런 다음, 압축된 **AzFilesHybrid.zip** PowerShell 모듈의 최신 버전을 다운로드하여 **C:\\Allfiles\\Labs\\02** 폴더에 압축을 풉니다(필요하면 폴더 만들기).
1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리자 권한으로 Windows PowerShell ISE**를 시작하고 **관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 값**이 3**인 Zone.Identifier** 대체 데이터 스트림을 제거**합니다. 이는 인터넷에서 다운로드되었음을** 나타냅니다.

   ```powershell
   Get-ChildItem -Path C:\Allfiles\Labs\02 -File -Recurse | Unblock-File
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 Azure 구독에 로그인합니다.

   ```powershell
   Connect-AzAccount
   ```

1. 메시지가 표시되면 이 랩에서 사용 중인 구독에서 소유자 역할을 사용하여 사용자 계정의 Microsoft Entra 자격 증명으로 로그인합니다.
1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 후속 스크립트를 실행하는 데 필요한 변수를** 설정합니다.

   ```powershell
   $subscriptionId = (Get-AzContext).Subscription.Id
   $resourceGroupName = 'az140-22-RG'
   $storageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName
   ```

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 작업의 앞부분에서 **만든 Azure Storage 계정을 나타내고 AD DS 인증을 구현하는 데 사용되는 AD DS 컴퓨터 개체를 만듭니**다.

   >**참고**: 이 스크립트 블록을 실행할 때 오류가 발생하는 경우 CopyToPSPath.ps1 스크립트와 동일한 디렉터리에 있는지 확인합니다. 이 랩의 앞부분에서 파일을 추출한 방법에 따라 AzFilesHybrid라는 하위 폴더에 있을 수 있습니다. PowerShell 컨텍스트에서 **cd AzFilesHybrid**를 사용하여 디렉터리를 폴더로 변경합니다.

   ```powershell
   Set-Location -Path 'C:\Allfiles\Labs\02'
   .\CopyToPSPath.ps1 
   Import-Module -Name AzFilesHybrid
   Join-AzStorageAccountForAuth `
      -ResourceGroupName $ResourceGroupName `
      -StorageAccountName $StorageAccountName `
      -DomainAccountType 'ComputerAccount' `
      -OrganizationalUnitDistinguishedName 'OU=WVDInfra,DC=adatum,DC=com'
   ```

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 스크립트 창에서 **다음을 실행하여 AD DS 인증이 Azure Storage 계정에서 사용하도록 설정되어 있는지 확인**합니다.

   ```powershell
   $storageaccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
   $storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties
   $storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions
   ```

1. 명령 `$storageAccount.AzureFilesIdentityBasedAuth.ActiveDirectoryProperties`의 출력이 스토리지 계정의 디렉터리 서비스를 나타내는 `AD`를 반환하는지 그리고 디렉터리 도메인 정보를 나타내는 `$storageAccount.AzureFilesIdentityBasedAuth.DirectoryServiceOptions` 명령의 출력이 다음 형식과 유사한지 확인합니다(`DomainGuid`, `DomainSid`, `AzureStorageSid`의 값은 다름).

   ```
   DomainName        : adatum.com
   NetBiosDomainName : adatum.com
   ForestName        : adatum.com
   DomainGuid        : 47c93969-9b12-4e01-ab81-1508cae3ddc8
   DomainSid         : S-1-5-21-1102940778-2483248400-1820931179
   AzureStorageSid   : S-1-5-21-1102940778-2483248400-1820931179-2109
   ```

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 Azure Portal을 표시하는 Microsoft Edge 창으로 전환하고, 스토리지 계정을 표시하는 블레이드에서 파일 공유**를 선택하고**, Active Directory** 설정이 **구성**되었는지 **확인**합니다.

   >**참고**: 변경 내용이 Azure Portal에 반영되려면 브라우저 페이지를 새로 고쳐야 할 수 있습니다.

#### 작업 4: Azure Files RBAC 기반 권한 구성

1. az140-dc-vm11에 대한 원격 데스크톱 세션 **내에서 Azure Portal을 표시하는 Microsoft Edge 창의 블레이드에서 이 연습의 앞부분에서 만든 스토리지 계정의 속성을 표시하는 블레이드의 왼쪽에 있는 **세로 메뉴의 데이터 스토리지** 섹션에서 파일 공유**를 선택합니다**.**
1. **파일 공유** 블레이드의 공유 목록에서 az140-22-profiles** 항목을 선택합니다**.
1. **az140-22-profiles** 블레이드의 왼쪽 세로 메뉴에서 액세스 제어(IAM)**를 선택합니다**.
1. 스토리지 계정의 **IAM(액세스 제어)** 블레이드에서 + 추가**를 선택하고 **드롭다운 메뉴에서 역할 할당** 추가를 선택합니다**. 
1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고 **검토 + 할당**을 선택합니다.

   |설정|값|
   |---|---|
   |역할|**Storage 파일 데이터 SMB 공유 기여자**|
   |다음에 대한 액세스 할당|**사용자, 그룹 또는 서비스 주체**|
   |선택|**az140-wvd-users**|

1. 스토리지 계정의 **IAM(액세스 제어)** 블레이드에서 + 추가**를 선택하고 **드롭다운 메뉴에서 역할 할당** 추가를 선택합니다**. 
1. **역할 할당 추가** 블레이드에서 다음 설정을 지정하고 **검토 + 할당**을 선택합니다.

   |설정|값|
   |---|---|
   |역할|**스토리지 파일 데이터 SMB 공유 관리자 권한 기여자**|
   |다음에 대한 액세스 할당|**사용자, 그룹 또는 서비스 주체**|
   |선택|**az140-wvd-admins**|

#### 작업 5: Azure Files 파일 시스템 권한 구성

1. az140-dc-vm11에 대한 **원격 데스크톱 세션 내에서 관리istrator: Windows PowerShell ISE** 창으로 전환**하고 관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 연습의 앞부분에서 **만든 스토리지 계정의 이름과 키를 참조하는 변수를 만듭니**다.

   ```powershell
   $resourceGroupName = 'az140-22-RG'
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0]
   $storageAccountName = $storageAccount.StorageAccountName
   $storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName).Value[0]
   ```

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 이 연습의 앞부분에서 만든 파일 공유에 대한 드라이브 매핑을 만듭니다.

   ```powershell
   $fileShareName = 'az140-22-profiles'
   net use Z: "\\$storageAccountName.file.core.windows.net\$fileShareName" /u:AZURE\$storageAccountName $storageAccountKey
   ```

1. **관리istrator: Windows PowerShell ISE** 콘솔에서 다음을 실행하여 현재 파일 시스템 권한을 확인합니다.

   ```powershell
   icacls Z:
   ```

   >**참고**: 기본적으로 NT 기관\\인증 사용자**와 **BUILTIN\\사용자** 모두 **사용자가 다른 사용자의 프로필 컨테이너를 읽을 수 있도록 허용하는 권한이 있습니다. 대신 해당 권한을 제거하고 필요한 최소 권한을 추가합니다.

1. **관리istrator: Windows PowerShell ISE** 스크립트 창에서 다음을 실행하여 최소 권한 원칙을 준수하도록 파일 시스템 권한을 조정합니다.

   ```powershell
   $permissions = 'ADATUM\az140-wvd-admins'+':(F)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'ADATUM\az140-wvd-users'+':(M)'
   cmd /c icacls Z: /grant $permissions
   $permissions = 'Creator Owner'+':(OI)(CI)(IO)(M)'
   cmd /c icacls Z: /grant $permissions
   icacls Z: /remove 'Authenticated Users'
   icacls Z: /remove 'Builtin\Users'
   ```

   >**참고**: 또는 파일 탐색기 사용하여 권한을 설정할 수 있습니다.
