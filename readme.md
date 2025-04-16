# AZ-140: Microsoft Azure Virtual Desktop 구성 및 작동

- **[랩 링크(HTML 형식)](https://microsoftlearning.github.io/AZ-140-Configuring-and-Operating-Microsoft-Azure-Virtual-Desktop/)**
- **MCT이신가요?** - [MCT를 위한 GitHub 사용자 가이드](https://microsoftlearning.github.io/MCT-User-Guide/)를 살펴보세요.
- **랩 지침을 수동으로 빌드해야 하나요?** - 지침은 [MicrosoftLearning/Docker-Build](https://github.com/MicrosoftLearning/Docker-Build) 리포지토리에서 확인할 수 있습니다.

## Microsoft의 역할

- 이 과정을 지원하려면 과정 콘텐츠를 자주 업데이트하여, 과정에서 사용하는 Azure 서비스를 이용해 콘텐츠를 최신 상태로 유지해야 합니다.  과정 작성자와 MCT 사이의 개방된 기여를 통해 Azure 플랫폼의 변경 내용을 적용하여 콘텐츠를 최신 상태로 유지할 수 있도록, 랩 지침과 랩 파일을 GitHub에 게시합니다.

- 이를 통해 이전에 경험하지 못한 방식으로 랩에 대한 협업을 경험하기 바랍니다. Azure가 변경하고 실시간 제공 중에 이를 먼저 발견하면 랩 소스에서 바로 개선 작업을 진행합니다.  동료 MCT를 도와주세요.

## 이러한 파일을 릴리스된 MOC 파일과 관련하여 어떻게 사용해야 하나요?

- 강사 핸드북과 PowerPoint는 여전히 과정 콘텐츠 교육의 기본 소스입니다.

- GitHub에서 이러한 파일은 학생 핸드북과 함께 사용하도록 설계되었지만, 중앙 리포지토리 역할을 하는 GitHub에 있기 때문에 MCT와 과정 작성자는 최신 랩 파일의 소스를 공유할 수 있습니다.

- 트레이너는 강의를 할 때마다 GitHub에서 최신 Azure 서비스를 지원하기 위해 변경된 내용을 확인하고 최신 파일을 가져와서 강의에 사용하는 것이 좋습니다.

## 학생 핸드북의 변경 사항은 어떻게 되나요?

- 학생 핸드북을 분기별로 검토하고 필요하다면 일반 MOC 릴리스 채널을 통해 업데이트합니다.

## 기고하려면 어떻게 해야 하나요?

- MCT라면 누구나 GitHub 리포지토리에서 코드 또는 콘텐츠에 대한 끌어오기 요청을 제출할 수 있으며, Microsoft와 과정 작성자는 필요한 경우 콘텐츠 및 랩 코드 변경 내용을 심사하고 포함합니다.

- 버그, 변경 내용, 개선 사항 및 아이디어를 제출할 수 있습니다.  우리보다 먼저 새 Azure 기능을 찾으셨나요?  새로운 데모를 제출해 주세요!

## 주의

> **중요**: Azure Virtual Desktop 구현을 위한 Microsoft Entra ID 기반 시나리오를 대상으로 랩이 업데이트되었습니다(이 랩에 대한 지침은 **지침** -&gt;**Labs_EntraID** 디렉터리에 있음). 

> **중요**: 다음 두 트랙은 더 이상 유지 관리되거나 지원되지 않으며 2025년 6월 1일에 삭제됩니다(이 랩에 대한 지침은 **지침** -&gt;**랩** 디렉터리에 있음)

- AD DS(Active Directory Domain Services) 이 트랙은 다음과 같은 랩으로 구성됩니다.

   - LAB_01L01_Prepare_for_deployment_of_AVD_ADDS.md
   - LAB_02L01_Deploy_host_pools_and_session_hosts_with_the_Azure_portal_ADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_ADDS.md
   - LAB_02L03_Deploy_host_pools_and_hosts_with_ARM_templates_ADDS.md
   - LAB_02L04_Deploy_and_manage_host_pools_and_hosts_with_PowerShell_ADDS.md
   - LAB_02L05_Create_and_manage_session_host_images_ADDS.md
   - LAB_03L01_Configure_Conditional_Access_policies_for_AVD_ADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_ADDS.md
   - LAB_04L02_Package_AVD_applications_ADDS.md
   - LAB_05L01_Implement_autoscaling_in_host_pools_ADDS.md

- Microsoft Entra Domain Services (Microsoft Entra DS). 이 트랙은 다음과 같은 랩으로 구성됩니다.

   - LAB_01L01_Prepare_for_deployment_of_AVD_AADDS.md
   - LAB_02L01_Create_and_configure_host_pools_and_session_hosts_AADDS.md
   - LAB_02L02_Implement_and_manage_storage_for_AVD_AADDS.md
   - LAB_04L01_Implement_and_manage_AVD_profiles_AADDS.md

### 강의 자료

MCT와 파트너는 이러한 자료에 액세스하여 학생에게 개별적으로 제공하는 것이 좋습니다.  진행 중인 수업의 일환으로 학생들에게 GitHub로 직접 이동하여 랩 단계에 액세스하게 하면, 과정에서 다른 UI에 액세스해야 하므로 학생들은 혼란을 겪게 됩니다. 별도의 랩 지침이 제공되는 이유를 학생들에게 설명하면 클라우드 기반 인터페이스와 플랫폼의 끊임없이 변하는 특성을 강조할 수 있습니다. Microsoft Learning은 GitHub의 파일 액세스를 지원하며, GitHub 사이트 탐색 지원은 이 과정을 가르치는 MCT에게만 제공됩니다.
