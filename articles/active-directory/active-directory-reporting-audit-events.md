<properties
   pageTitle="Azure Active Directory 稽核報告事件 | Microsoft Azure"
   description="可供檢視以及從 Azure Active Directory 下載的稽核事件"
   services="active-directory"
   documentationCenter=""
   authors="dhanyahk"
   manager="mbaldwin"
   editor=""/>

<tags
   ms.service="active-directory"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="identity"
   ms.date="09/19/2016"
   ms.author="dhanyahk"/>

# Azure Active Directory 稽核報告事件

*這份文件是 [Azure Active Directory 報告指南](active-directory-reporting-guide.md)的一部分。*

Azure Active Directory 稽核報告可協助客戶識別其 Azure Active Directory 中發生的特殊權限動作。特殊權限動作包括提高權限變更 (例如，角色建立或密碼重設)、原則設定變更 (例如密碼原則) 或目錄設定變更 (例如，網域同盟設定變更)。報告會提供的稽核記錄包括事件名稱、執行動作的執行者、受變更影響的目標資源，以及日期和時間 (UTC)。客戶能透過 [Azure 入口網站](https://portal.azure.com/)擷取其 Azure Active Directory 的稽核事件清單，如[檢視您的稽核記錄](active-directory-reporting-azure-portal.md)中所述。


## 稽核報告事件清單
<!--- audit event descriptions should be in the past tense --->

事件 | 事件說明
------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------
**使用者事件** |
新增使用者 | 將使用者新增至目錄。
刪除使用者 | 刪除目錄中的使用者。
設定授權屬性 | 設定目錄中使用者的授權屬性。
重設使用者密碼 | 重設目錄中使用者的密碼。
變更使用者密碼 | 變更目錄中使用者的密碼。
變更使用者授權 | 變更指派給目錄中使用者的授權。若要查看哪些授權已更新，請查看下面的[更新使用者](#update-user-attributes)屬性
更新使用者 | 更新目錄中的使用者。關於可更新的屬性，[請參閱下方](#update-user-attributes)。
設定強制變更使用者密碼 | 設定可強制使用者在登入時變更其密碼的屬性。
更新使用者認證 | 使用者變更密碼  
**群組事件** |
新增群組 | 在目錄中建立群組。
更新群組 | 更新目錄中的群組。若要查看哪些群組屬性已更新，請參考下一節中的[稽核的群組屬性](#update-group-attributes)
刪除群組 | 從目錄刪除群組。
將成員加入群組 | 將成員加入目錄中的群組。
從群組移除成員 | 將成員從目錄中的群組移除。
CreateGroupSettings | 建立群組設定
UpdateGroupSettings | 更新群組設定。若要查看哪些群組設定已更新，請參考下一節中的[稽核的群組屬性](#update-group-attributes)
DeleteGroupSettings | 刪除群組設定
SetGroupLicense | 設定群組授權。
SetGroupManagedBy | 將群組設定為由使用者管理
AddGroupMember | 將成員新增至群組
RemoveGroupMember | 從群組移除成員
AddGroupOwner | 將擁有者新增至群組
RemoveGroupOwner | 移除群組中的擁有者
**應用程式事件** |
新增服務主體 | 將服務主體新增至目錄。
移除服務主體 | 移除目錄中的服務主體
新增服務主體認證 | 將認證新增至服務主體。
移除服務主體認證 | 移除服務主體的認證。
新增委派項目 | 已在目錄中建立 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity)。
設定委派項目 | 已在目錄中更新 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity)。
移除委派項目 | 已在目錄中刪除 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#oauth2permissiongrant-entity)。
**角色事件** |
將角色成員新增至角色 | 將使用者新增至目錄角色。
移除角色的角色成員 | 從目錄角色移除使用者。
設定公司連絡人資訊 | 設定公司層級的連絡人喜好設定。這包括用於行銷的電子郵件地址，以及 Microsoft Online Services 的相關技術通知。
新增委派項目 | 已在目錄中建立 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity)。
設定委派項目 | 已在目錄中更新 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity)。
移除委派項目 | 已在目錄中刪除 [OAuth2PermissionGrant](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#OAuth2PermissionGrantEntity)。
AddSevicePrincipalOwner | 將擁有者新增至服務主體。
RemoveSevicePrincipalOwner | 移除服務主體中的擁有者。
AddApplication | 新增應用程式。
UpdateApplication | 更新應用程式。若要查看哪些應用程式設定已更新，請參考下一節中的[稽核的應用程式屬性](#update-application-attributes)
DeleteApplication | 刪除應用程式。
RestoreApplication|還原應用程式。
AddApplicationOwner | 	將擁有者新增至應用程式。
RemoveApplicationOwner | 	移除應用程式中的擁有者。
**角色事件** |
將角色成員新增至角色 | 將使用者新增至目錄角色。
移除角色的角色成員 | 從目錄角色移除使用者。
AddRoleDefinition | 新增角色定義。
UpdateRoleDefinition | 更新角色定義。若要查看哪些角色設定已更新，請參考下一節中的[稽核的角色定義屬性](#update-role-definition-attributes)
DeleteRoleDefinition | 刪除角色定義。
AddRoleAssignmentToRoleDefinition | 將角色指派新增至角色定義。
RemoveRoleAssignmentFromRoleDefinition | 移除角色定義中的角色指派。
AddRoleFromTemplate | 從範本新增角色。
UpdateRole | 更新角色。
AddRoleScopeMemberToRole | 將範圍成員新增至角色。
RemoveRoleScopedMemberFromRole | 移除角色中的範圍成員。
**裝置事件 (所有新事件)** |
AddDevice | 新增裝置。
UpdateDevice | 更新裝置。若要查看哪些裝置屬性已更新，請參考下一節中的[稽核的裝置屬性](#update-device-attributes)
DeleteDevice | 刪除裝置。
AddDeviceConfiguration | 新增裝置組態。
UpdateDeviceConfiguration | 更新裝置組態。若要查看哪些裝置組態屬性已更新，請參考下一節中的[稽核的裝置組態屬性](#update-device-configuration-attributes)
DeleteDeviceConfiguration | 刪除裝置組態。
AddRegisteredOwner | 將已註冊的擁有者新增至裝置。
AddRegisteredUsers | 將已註冊的使用者新增至裝置。
RemoveRegisteredOwner | 移除裝置中已註冊的擁有者。
RemoveRegisteredUsers | 移除裝置中已註冊的使用者。
RemoveDeviceCredentials | 移除裝置認證。
**B2B 事件** |
已上傳的 Batch 邀請。 | 系統管理員已上傳的檔案中，包含將傳送給合作夥伴使用者的邀請。
已處理的 Batch 邀請。 | 已處理包含合作夥伴使用者邀請的檔案。
邀請外部使用者。 | 已邀請外部使用者至目錄。
兌換外部使用者的邀請。 | 外部使用者已兌換其至目錄的邀請。
新增外部使用者至群組。 | 已將目錄中群組的成員資格指派給外部使用者。
指派外部使用者至應用程式。 | 已將應用程式的直接存取權指派給外部使用者。
建立熱門租用戶 | 已透過邀請兌換在 Azure AD 中建立新的租用戶。
建立熱門使用者。 | 已透過邀請兌換在 Azure AD 中為現有的租用戶建立使用者。
**管理單位 (所有新事件)** |
AddAdministrativeUnit | 新增管理單位。
UpdateAdministrativeUnit| 	更新管理單位。若要查看哪些管理單位屬性已更新，請參考下一節中的[稽核的管理單位屬性](#update-administrative-unit-attributes)
DeleteAdministrativeUnit| 	刪除管理單位。
AddMemberToAdministrativeUnit| 	將成員新增至管理單位。
RemoveMemberFromAdministrativeUnit| 	移除管理單位中的成員。
**目錄事件** |
將夥伴新增至公司 | 將夥伴新增至目錄。
移除公司的夥伴 | 移除目錄中的夥伴。
DemotePartner | 將夥伴降級。
將網域新增至公司 | 將網域新增至目錄。
移除公司的網域 | 移除目錄中的網域。
更新網站 | 更新目錄上的網域。若要查看哪些網域屬性已更新，請參考下一節中的[稽核的網域屬性](#update-domain-attributes)
設定網域驗證 | 變更公司的預設網域設定。
設定公司連絡人資訊 | 設定公司層級的連絡人喜好設定。這包括用於行銷的電子郵件地址，以及 Microsoft Online Services 的相關技術通知。
設定網域的同盟設定 | 更新網域的同盟設定。
驗證網域 | 驗證目錄上的網域。
驗證電子郵件驗證的網域 | 使用電子郵件驗證來驗證目錄上的網域。
對公司設定 DirSyncEnabled 旗標 | 設定可針對 Azure AD Sync 啟用目錄的屬性。
設定密碼原則 | 設定使用者密碼的長度和字元限制。
設定公司資訊 | 更新公司層級資訊。如需詳細資訊，請參閱 [Get-MsolCompanyInformation](https://msdn.microsoft.com/library/azure/dn194126.aspx) PowerShell Cmdlet。
SetCompanyAllowedDataLocation | 	設定公司允許的資料位置。
SetCompanyDirSyncEnabled | 設定 DirSyncEnabled 旗標。
SetCompanyDirSyncFeature | 	設定 DirSync 功能。
SetCompanyInformation | 設定公司資訊。
SetCompanyMultiNationalEnabled | 	設定啟用公司多語系功能。
SetDirectoryFeatureOnTenant | 	在租用戶上設定目錄功能。
SetTenantLicenseProperties | 設定租用戶授權屬性。
CreateCompanySettings | 建立公司設定
UpdateCompanySettings | 更新公司設定。若要查看哪些公司屬性已更新，請參考下一節中的[稽核的公司屬性](#update-company-attributes)
DeleteCompanySettings | 刪除公司設定
SetAccidentalDeletionThreshold | 	設定意外刪除臨界值。
SetRightsManagementProperties | 設定 Rights Management 屬性。
PurgeRightsManagementProperties | 清除 Rights Management 屬性。
UpdateExternalSecrets | 更新外部密碼
**原則事件 (所有新事件)** |
AddPolicy | 	新增原則。
UpdatePolicy |	更新原則。
DeletePolicy |	刪除原則。
AddDefaultPolicyApplication |	將原則新增至應用程式。
AddDefaultPolicyServicePrincipal |	將原則新增至服務主體。
RemoveDefaultPolicyApplication |	移除應用程式中的原則。
RemoveDefaultPolicyServicePrincipal |	移除服務主體中的原則。
RemovePolicyCredentials |	移除原則認證。

## 稽核報告保留
Azure AD 稽核報告中的事件會保留 180 天。如需保留報告的詳細資訊，請參閱 [Azure Active Directory 報告保留原則](active-directory-reporting-retention.md)。

若是對於儲存其稽核事件較長的保留時間感興趣的客戶，報告 API 可用來定期將稽核事件提取至不同的資料存放區。如需詳細資訊，請參閱[開始使用報告 API](active-directory-reporting-api-getting-started.md)。

## 每個稽核事件包含的屬性

屬性 | 說明
------------- | --------------------------------------------------------------
日期和時間 | 發生稽核事件的日期和時間
執行者 | 執行此動作的使用者或服務主體
動作 | 已執行的動作
目標 | 被執行此動作的使用者或服務主體


## 「更新使用者」屬性
「更新使用者」稽核事件包含有關哪些使用者屬性已更新的其他資訊。每個屬性的舊值和新值都包含在內。

屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AccountEnabled | 使用者可以登入。
AssignedLicense | 指派給使用者的所有授權。
AssignedPlan | 從指派給使用者的授權產生的服務方案。
LicenseAssignmentDetail | 指派給使用者的授權詳細資訊。例如，如果涉及以群組為基礎的授權，這會包含授與授權的群組。
行動 | 使用者的行動電話。
OtherMail | 使用者的備用電子郵件地址。
OtherMobile | 使用者的備用行動電話。
StrongAuthenticationMethod | 使用者針對多因素驗證所設定的驗證方法清單，例如來自行動應用程式的語音電話、簡訊或驗證碼。
StrongAuthenticationRequirement | 是否對此使用者強制使用、啟用或停用多因素驗證。
StrongAuthenticationUserDetails | 使用者用於多因素驗證和密碼重設驗證的電話號碼、備用電話號碼和電子郵件地址。
StrongAuthenticationPhoneAppDetail | 為了執行 2FA 登入而註冊的電話應用程式的詳細資料
TelephoneNumber | 使用者的電話號碼。
AlternativeSecurityId | 物件的替代安全性識別碼。
CreationType |使用者的建立方法 (經由邀請或病毒)。
InviteTicket |使用者的邀請票證清單。
InviteReplyUrl |接受邀請時所要回覆的 url 清單。
InviteResources |使用者受邀的資源清單。
LastDirSyncTime |上次因為從權威性 (客戶、內部部署) 目錄同步處理而更新物件的時間。
MSExchRemoteRecipientType |對應至 MSO 收件者類型。請參考 [MSO 收件者類型] https://msdn.microsoft.com/library/microsoft.office.interop.outlook.recipient.type.aspx 以取得收件者類型
PreferredDataLocation |使用者、群組、連絡人、公用資料夾或裝置的資料偏好的位置。
ProxyAddresses |在外部郵件系統中用以辨識 Exchange Server 收件者物件的位址。
StsRefreshTokensValidFrom |傳達重新整理權杖撤銷資訊 - 任何在此時間之前發出的 STS 重新整理權杖都會被視為到期。
UserPrincipalName |UPN 是使用者的網際網路登入名稱。
UserState |使用只 PendingApproval/PendingAcceptance/Accepted/PendingVerification 的狀態。
UserStateChangedOn |UserState 上次變更的 TimeStamp。用來觸發生命週期工作流程。
UserType |使用者類型。成員 (0)、來賓 (1)、病毒 (2)。

## 「更新群組」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
分類 | 統一群組的分類 (HBI、MBI 等)。
說明 | 人們可讀取的物件相關描述性語句。  
DisplayName |物件的顯示名稱
DirSyncEnabled |表示同步處理是否從權威性 (客戶、內部部署) 目錄發生。
GroupLicenseAssignment |群組的授權指派
GroupType |群組類型，統一 (0)
IsMembershipRuleLocked |表示 MembershipRule 屬性是由自助式群組管理服務所設定，使用者無法加以變更。只適用於 GroupType 包含 GroupType.DynamicMembership 的群組
IsPublic |用來表示群組為公開/私密的旗標。
LastDirSyncTime |上次從權威性 (客戶、內部部署) 目錄同步處理因而更新物件的時間。
Mail |主要電子郵件地址
MailEnabled |表示此群組是否有電子郵件功能。
MailNickname |通訊錄物件的 Moniker，通常為其電子郵件名稱前面加上 "@" 符號的部分。   
MembershipRule |字串，表示自助式群組管理服務用來判斷哪些成員應屬於此群組的準則。另請參閱 IsMembershipRuleLocked。只適用於 GroupType 包含 GroupType.DynamicMembership 的群組。
MembershipRuleProcessingState| 自助式群組管理服務所定義的列舉值，用來定義此群組的成員資格處理狀態。只適用於 GroupType 包含 GroupType.DynamicMembership 的群組。
ProxyAddresses |在外部郵件系統中用以辨識 Exchange Server 收件者物件的位址。
RenewedDateTime |最近更新群組時的時間戳記記錄。   
SecurityEnabled |表示群組中的成員資格是否可能影響授權決策。
WellKnownObject |標示目錄物件，並將它指定為其中一個預先定義的集合。

## 「更新裝置」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AccountEnabled | 表示是否可以驗證安全性主體。
CloudAccountEnabled | 表示是否可以驗證安全性主體。當裝置在內部部署主控時由 InTune 寫入。
CloudDeviceOSType | 以作業系統 (如 Windows RT、iOS) 為基礎的裝置類型。由雲端服務 (例如 Intune) 設定時，此屬性在 DeviceOSType 的目錄中具有權威性。
CloudDeviceOSVersion | OS 版本。由雲端服務 (例如 Intune) 設定時，此屬性在 DeviceOSVersion 的目錄中具有權威性。
CloudDisplayName | displayName LDAP 屬性的值。由雲端服務 (例如 Intune) 設定時，此屬性在 displayName 的目錄中具有權威性。
CloudCreated |表示物件是否由雲端服務建立。
CompliantUntil | 等到何時裝置被視為相容。
DeviceMetadata |裝置的自訂中繼資料
DeviceObjectVersion | 此屬性用來識別裝置的結構描述版本。
DeviceOSType |以作業系統 (如 Windows RT、iOS) 為基礎的裝置類型。由註冊服務寫入，且預定由 MDM 管理服務或 STS 輕型管理服務更新。
DeviceOSVersion |OS 版本。由註冊服務寫入，且預定由 MDM 管理服務或 STS 輕型管理服務更新。
DevicePhysicalIds |多重值的屬性，預定用來儲存實體裝置的識別碼。這可能包括 BIOS 識別碼、TPM 指紋、硬體特定識別碼等。
DirSyncEnabled | 表示同步處理是否從權威性 (客戶、內部部署) 目錄發生。
DisplayName |物件的顯示名稱
IsCompliant | 此屬性用來管理裝置的行動裝置管理狀態。
IsManaged |此屬性用來表示雲端 MDM 所管理的裝置。
LastDirSyncTime |上次因為從權威性 (客戶、內部部署) 目錄同步處理而更新物件的時間。

## 「更新裝置組態」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
MaximumRegistrationInactivityPeriod |在考慮移除之前，裝置可維持非使用中的天數上限。
RegistrationQuota |用來限制單一使用者允許之裝置註冊數目的原則。

## 「更新服務主體組態」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AccountEnabled |表示是否可以驗證安全性主體。
AppPrincipalId | 安全性主體外部的身分識別 (由應用程式定義)。
DisplayName |物件的顯示名稱
ServicePrincipalName | 服務主體名稱，其中包含「名稱/授權單位」，其中名稱指定應用程式類別值，而授權單位至少包含可指定服務主體識別項的主機名稱[:連接埠] 或「名稱」。

## 「更新應用程式」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AppAddress |指派給服務主體的位址 (重新導向 URL) 集。
AppId |應用程式的應用程式識別碼   
AppIdentifierUri | 應用程式 URI，用來識別應用程式。這通常是應用程式存取 URL。
AppLogoUrl |儲存在 CDN 中的應用程式標誌影像的 url。
AvailableToOtherTenants | 應用程式為多租用戶應用程式 (也就是可供其他租用戶使用) 則為 True。
DisplayName | 應用程式名稱的顯示名稱
Entitlement |應用程式權利的清單。
ExternalUserAccountDelegationsAllowed |表示資源應用程式是否為受信任資源的旗標，可以為外部使用者帳戶建立委派項目。
GroupMembershipClaims |群組成員資格宣告原則。
PublicClient | 如果用戶端無法保持祕密狀態 (也就是 OAuth2.0 中的非機密用戶端) 則為 true
RecordConsentConditions | 同意條件的類型，如合約條款所定義︰無 (0)、SilentConsentForPartnerManagedApp (1)。此值會在圖形 API 結構描述中公開，只可由租用戶系統管理員設定/變更。
RequiredResourceAccess |RequiredResourceAccess 屬性值的 XML 內容。
WebApp |如果為 true，表示此應用程式是 Web 應用程式。
WwwHomepage |主要網頁。

## 「更新角色」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AppAddress |指派給服務主體的位址 (重新導向 URL) 集。
BelongsToFirstLoginObjectSet |如果為 true，表示此物件屬於讓新租用戶的第一個系統管理員能夠登入所需的物件集。
Builtin |表示物件的存留期是否由系統所擁有。
說明 | 人們可讀取的物件相關描述性語句。  
DisplayName |物件的顯示名稱
MailNickname | 通訊錄物件的 Moniker，通常為其電子郵件名稱前面加上 "@" 符號的部分。
RoleDisabled |表示是否應該為了存取檢查的目的而忽略角色。
RoleTemplateId | 角色範本的身分識別。
ServiceInfo |服務特定佈建資訊，可供 MOAC 和/或其他服務執行個體 (相同或不同的服務類型) 取用。
TaskSetScopeReference |識別 TaskSet 和一組與 Role 或 RoleTemplate 相關聯的範圍。
ValidationError |同盟服務所發佈的資訊，描述有關屬性的非暫時性、服務特定錯誤，或用來解決的物件管理員動作連結。
WellKnownObject |標示目錄物件，並將它指定為其中一個預先定義的集合。

## 「更新角色定義」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AssignableScopes |將此 RoleDefinition 指派給安全性主體時，可參考的授權範圍集合。
DisplayName |物件的顯示名稱
GrantedPermissions |此 RoleDefinition 所授與的權限。


## 「更新管理單位」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
說明 |	此屬性會在您變更管理單位的描述時更新。
DisplayName |	此屬性會在您變更管理單位的名稱時更新。

## 「更新公司」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
AllowedDataLocation | 允許佈建公司使用者的位置。
AuthorizedServiceInstance | 可能部署方案的服務執行個體名稱。
DirSyncEnabled |表示同步處理是否從權威性 (客戶、內部部署) 目錄發生。
DirSyncStatus |表示此租用戶內容中的通訊錄物件是否從權威性 (客戶、內部部署) 目錄發生；公司物件上 DirSyncEnabled 屬性的擴充功能。
DirSyncFeatures |位元旗標，用來追蹤租用戶的已啟用和停用的目錄同步處理功能。
DirectoryFeatures |已啟用/停用的目錄功能。
DirSyncConfiguration |包含目前租用戶特定的所有 DirSync 組態。
DisplayName |物件的顯示名稱
IsMnc |如果公司已啟用多語系公司功能則設定為 "true" 的布林值旗標。
ObjectSettings | 適用於物件範圍的設定集合。
PartnerCommerceUrl |合作夥伴的商務網站 URL。
PartnerHelpUrl |合作夥伴的說明網站 URL。
PartnerSupportEmail | 合作夥伴的支援電子郵件 URL。
PartnerSupportTelephone |合作夥伴的支援電話 URL。
PartnerSupportUrl |合作夥伴的支援網站 URL。
StrongAuthenticationDetails |StrongAuthentication 相關詳細資料。
StrongAuthenticationPolicy |公司的強式驗證原則。
TechnicalNotificationMail |用來通知公司相關技術問題的電子郵件地址。
TelephoneNumber |符合 ITU Recommendation E.123 的電話號碼。
TenantType |租用戶的類型。如果未指定此值，則租用戶是公司。否則，可能的值為︰MicrosoftSupport (0)、SyndicatePartner (1)、BreadthPartner(2)、BreadthPartnerDelegatedAdmin (3)、ResellerPartnerDelegatedAdmin (4)、ValueAddedResellerPartnerDelegatedAdmin (5)。
VerifiedDomain |一組繫結至公司的 DNS 網域名稱。

## 「更新網域」屬性
屬性 | 說明
-------------------------------     | -------------------------------------------------------------------------------------------------------------------------------------------------------
功能 |描述網域功能 (如果有的話) 的位元旗標。
預設值 |表示網域是否為預設值；例如，當系統管理員在 MOAC 中建立新使用者時的預設 UserPrincipalName 尾碼。
Initial |表示網域是否為公司的初始網域 (如 OCP 所配置)。初始網域是 Microsoft Online 網域的唯一子網域，例如 contoso3.microsoftonline.com。
LiveType |對應 Windows Live 命名空間 (如果有的話) 的類型。
名稱 | 端點的識別碼。
PasswordNotificationWindowDays |在密碼到期前使用者接獲通知的天數。
PasswordValidityPeriodDays | 密碼在必須變更前的理想天數。

稽核記錄是許多標準規定的必要控制項。對於使用 Azure Active Directory 稽核報告以符合其標準規定的客戶，建議客戶提交一份本說明主題的副本 (連同客戶匯出的稽核報告副本)，協助說明報告的詳細資料。如果稽核人員想要了解 Azure 目前符合的法規，請將稽核人員導向至 Microsoft Azure 信任中心的[規範](https://azure.microsoft.com/support/trust-center/compliance/)頁面。

<!---HONumber=AcomDC_0921_2016-->