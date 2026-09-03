# WindowsStore.admx — Inverted semantics for "Disable all apps from Microsoft Store"

## Summary

The **"Disable all apps from Microsoft Store"** Group Policy setting has counter-intuitive and inverted `Enabled` / `Disabled` semantics.

The policy name indicates that enabling the policy should disable Microsoft Store applications.

However, the actual behavior is the opposite:

|Policy state|`DisableStoreApps` value|Actual behavior|
|---|--:|---|
|Enabled|`0`|Microsoft Store applications are allowed|
|Disabled|`1`|Microsoft Store applications are disabled|

The policy help text correctly describes this behavior, so this does not appear to be a documentation mismatch or a localization issue.

The issue is that the **policy name itself contradicts the semantics of the Enabled and Disabled states**.

---

## Affected policy (see `evidence` folder)

**ADMX file:** `WindowsStore.admx`

**Policy name:**

```text
DisableStoreApps
```

**English display name:**

```text
Disable all apps from Microsoft Store
```

**French display name:**

```text
Désactiver toutes les applications du Microsoft Store
```

**Group Policy path:**

```text
Computer Configuration
└── Administrative Templates
    └── Windows Components
        └── Store
            └── Disable all apps from Microsoft Store
```

**Registry location:**

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\WindowsStore
```

**Registry value:**

```text
DisableStoreApps
```

---

## Current behavior

When the policy is configured as:

```text
Enabled
```

the following value is configured:

```text
DisableStoreApps = 0
```

Microsoft Store applications are therefore allowed to launch.

When the policy is configured as:

```text
Disabled
```

the following value is configured:

```text
DisableStoreApps = 1
```

Microsoft Store applications are prevented from launching.

This includes applications that were preinstalled or installed from Microsoft Store such as the new Windows Terminal, notepad or the windows calculator.

---

## Why this is confusing


For a policy named:

```text
Disable all apps from Microsoft Store
```

The expected behavior is :

```text
Policy Enabled  -> Disable Microsoft Store applications
Policy Disabled -> Do not disable Microsoft Store applications
```

Instead, the policy behaves as:

```text
Policy Enabled  -> Allow Microsoft Store applications
Policy Disabled -> Disable Microsoft Store applications
```

The administrator therefore has to read the policy help text to discover that the meaning of `Enabled` and `Disabled` is inverted relative to the policy name.

This is especially error-prone when reviewing or configuring a large number of Group Policy settings.

---

## Expected behavior

Ideally, the semantics should match the policy name:

```text
Policy Enabled  -> Disable Microsoft Store applications
Policy Disabled -> Allow Microsoft Store applications
```

However, changing the existing registry mapping could introduce compatibility issues for environments already using this policy.

A safer solution may therefore be to **rename the policy using positive semantics**, while preserving the existing registry behavior.

For example:

```text
Allow all apps from Microsoft Store
```

with:

```text
Policy Enabled  -> DisableStoreApps = 0
Policy Disabled -> DisableStoreApps = 1
```

This would make the UI semantics consistent without changing the underlying registry behavior.

---

## Steps to reproduce

1. Open `gpedit.msc`.
    
2. Navigate to:
    

```text
Computer Configuration
> Administrative Templates
> Windows Components
> Store
```

3. Open:
    

```text
Disable all apps from Microsoft Store
```

4. Set the policy to **Enabled**.
    
5. Apply Group Policy.
    
6. Check:
    

```powershell
Get-ItemPropertyValue `
    -Path 'HKLM:\SOFTWARE\Policies\Microsoft\WindowsStore' `
    -Name 'DisableStoreApps'
```

The resulting value is:

```text
0
```

7. Try opening a Microsoft store application such as Notepad --> Not Working.

8. Set the same policy to **Disabled** and apply Group Policy again.

The resulting value becomes:

```text
1
```

The value `1` disables applications originating from Microsoft Store.

---

## Microsoft documentation

The official help for that policy is :
```text
Disable turns off the launch of all apps from the Microsoft Store that came pre-installed or were downloaded. Apps will not be updated. Your Store will also be disabled. Enable turns all of it back on. This setting applies only to Enterprise and Education editions of Windows.
```

Therefore, the observed behavior is consistent with the documented implementation.

The issue is specifically the **inconsistent naming and Group Policy UI semantics**.

---

## Localization

This issue is **not specific to the French localization**.

Both the English and French policy names use negative wording:

```text
EN: Disable all apps from Microsoft Store
FR: Désactiver toutes les applications du Microsoft Store
```

while retaining the same inverted Enabled/Disabled semantics.

This indicates that the issue originates from the policy design rather than from the `fr-FR` ADML translation.

---
## Source Files

The affected administrative template files were taken directly from the local Windows installation : 

```text
C:\Windows\PolicyDefinitions\WindowsStore.admx C:\Windows\PolicyDefinitions\en-US\WindowsStore.adml
```

### SHA256

```text
0469E4FC47CDF89FCC3B8EEC0869BC27C74C991C460D08EFE635B3E7423EC3DC       C:\Windows\PolicyDefinitions\WindowsStore.admx

1F877B7A94FF8B235C5BA3C821896136D8E48E720B8749E2B4DEA39DEFAC0405       C:\Windows\PolicyDefinitions\en-US\WindowsStore.adml
```

### Files infos

```text
C:\Windows\PolicyDefinitions\WindowsStore.admx         5248 01/04/2024 09:22:09
C:\Windows\PolicyDefinitions\en-US\WindowsStore.adml   3584 01/04/2024 18:12:33
```

---
## Impact

This issue can result in administrators unintentionally disabling Microsoft Store applications.

Possible consequences include:

- Microsoft Store applications failing to launch;
    
- packaged Windows applications unexpectedly becoming unavailable;
    
- administrative troubleshooting caused by an unintentionally applied restriction;
    
- incorrect configuration in security hardening guides;
    
- incorrect interpretation by compliance or configuration-management processes relying on the displayed Group Policy name.
    

The behavior is particularly misleading because selecting **Disabled** on a policy named **"Disable all apps from Microsoft Store"** actually applies the restriction described by the policy name.

---

## Tested environment

Observed on:

```text
Edition: Windows 11 Enterprise
Version: 25H2
Build: 26200

Edition: Windows 10 Enterprise
Version : 20h2

Edition Windows Server 2025
```

The policy is documented by Microsoft as applying to supported Enterprise and Education editions.

---

## Proposed classification

```text
Component: Windows Administrative Templates / Group Policy
ADMX: WindowsStore.admx
Policy: DisableStoreApps
Type: Policy naming / semantic inconsistency
Severity: Low
Impact: Administrative configuration error / misleading UI
```