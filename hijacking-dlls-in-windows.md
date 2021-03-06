---
title: 'Hijacking DLLs in Windows'
author: Wietze Beukema
date: October 2020
title-background: img/dlls.png
title-colour: one
---

# Hello, who dis?

### Wietze Beukema

* Threat Hunting
* Detection Engineering & Research
* PwC UK, London


<br /><br />
Keep in touch:

![social](img/twitter.svg) [\@Wietze](https://www.twitter.com/wietze)

![social](img/github.svg) [github.com/wietze](https://github.com/wietze)

![social](img/email.svg) [wietze.beukema@pwc.com](mailto:wietze.beukema@pwc.com)

# 👾 DLL Hijacking

# DLL Hijacking

"Tricking a legitimate/trusted application into loading an arbitrary DLL"

# What's in a name?

![_](img/name.png){.full-width-title}


# DLL Hijacking (2)
- **T1574**: Hijack Execution Flow
  - **T1574.001**: DLL Search Order Hijacking
  - **T1574.002**: DLL Side-Loading
- **T1218**: Signed Binary Proxy Execution (?)
- **T1036**: Masquerading


![](img/attack.png)

# DLL Hijacking (3)
## But why?

- Your evil code, executed by a trusted executable
- Great for Execution, Persistence, Privilege Escalation
- The threat is real




# Types{.emphasis}
## A few common ones

|Type | Method |
|---- | ----------- |
|1: **DLL replacement** | Replace a legitimate DLL with the evil DLL. |
|2: **DLL search order hijacking** | Place the evil DLL in a location that is ched for before the legitimate DLL. |
|3: **Phantom DLL Hijacking** | Drop the evil DLL in place of a missing/existing DLL that a legitimate application tries to load. |
|4: **DLL redirection** | Change the location in which the DLL is searched for, by editing the %PATH% environment variable or .exe.manifest / .exe.local. |
|5: **WinSxS DLL replacement** (_'DLL side-loading'_)| Replace the legitimate with the evil DLL in the relevant WinSxS folder of the targeted DLL. |
|6: **Relative path DLL Hijacking** | Copy (and optionally rename) the legitimate application to a user-writeable folder, alongside the evil DLL. |


# Types
## 1: DLL replacement

> "Replace a legitimate DLL with the evil DLL."

* Most basic type
* Could run DLL with high integrity (although replacement usually requires this too)
* Often used together with DLL Proxying [[1]].

### Example{.emphasis}
Stuxnet: renamed legitimate Siemens DLL `S7OTBXDX.DLL` in `c:\windows\` and added its own malicious version, implementing all exports but with modified logic when interacting with the PLC [[2]].  _Execution_ + _Persistence_!

![](img/dll.png){.split}

# Types
## 2: DLL search order hijacking (T1574.001)

> "Place the evil DLL in a location that is searched for before the legitimate DLL."

* Look at the _DLL search order_: (1) Directory of application → (2) System32 directory → (3) System directory → (4) Windows directory → (5) Current directory → (6) `%PATH%`
* CVE potential: especially executables running with high integrity
* Good way to find these: look for `FILE NOT FOUND` in Procmon

### Example{.emphasis}
Vault7 Leaks: portable Chrome tries to load `DWrite.dll`, located in `System32\`. Because no absolute path is specified, the directory of the application is tried first [[3]]. _Execution_ (+ _Persistence_)!


# Types
## 3: Phantom DLL Hijacking

> "Drop the evil DLL in place of a missing/non-existing DLL that a legitimate application tries to load."

* More rare, yet not uncommon
* Less likely your DLL will break the running application
* CVE potential

### Example{.emphasis}

Vault7 leaks: Kaspersky process `avp.exe` intended to load `WHEAPGRD.DLL`. Due to a bug, it would prepend the drive letter, meaning it would try to load non-existing file `CWHEAPGRD.DLL` [[4]]. _Execution_ + _Persistence_ (+ _Elevation?_)

# Types
## 4: DLL redirection
> "Change the location in which the DLL is searched for, by editing %PATH% or .exe.manifest / .exe.local."

# Types
## 5: WinSxS DLL replacement (T1574.002)
> "Replace the legitimate with the evil DLL in the relevant WinSxS folder of the targeted DLL."

* Sometimes referred to as 'DLL Side-Loading' [[5]]

![](img/sideloading.png)


# Types
## 6: Relative path DLL Hijacking
> "Copy (and optionally rename) the legitimate application to a user-writeable folder, alongside the evil DLL."


::: incremental
* Simple: DLL Hijacking with least preconditions
* No CVE/LOLbin potential, yet very powerful
* Combine with other techniques for more impact, e.g. UAC bypass

### Example{.emphasis}
PwC Threat Intelligence observed a dropper ('xStart') used by a threat actor targetting Chinese organisations to deliver Cobalt Strike payloads. Through phishing, a user is tricked into opening a legitimate (but renamed) `winword.exe` that is put alongside a malicious alongside a malicious `wwlib.dll`. When the renamed `winword.exe` is executed, `wwlib.dll` is triggered, starting phase 2 of the attack [[6]]. _Execution_!



![](img/vt.png){.split}
:::

# Types
## 6: Relative path DLL Hijacking (2)
> "Copy (and optionally rename) the legitimate application to a user-writeable folder, alongside the evil DLL."

* Nearly 300 (!) `system32\` executables are vulnerable to this [[7]]

* Many, many more outside this dir, as well as non-Microsoft (see Hexacorn et al.)

* Can be combined with UAC bypass techniques


![](img/procmon.png){.center}

# Types
## 6: Relative path DLL Hijacking (3)
> "Copy (and optionally rename) the legitimate application to a user-writeable folder, alongside the evil DLL."

This poses a problem:

  * Relies on legitimate executables
  * Many possible candidates
  * Actively used in the wild
  * Impact is medium to high

So how to prevent / detect this?


![](img/blog.png){.split}


# 🧭 Prevention & Detection

# Preventing DLL Hijacking

+---------------------------------+---------------------------+
|**🔧 Developers**                 | **👩‍💻 👨‍💻 System admins**|
+=================================+=================================+
|  Use absolute DLL paths         |  Use  |
|   where possible<br />          |   `PreferSystem32Images` |
|      _e.g. _`system32\`_        |  and `MicrosoftSignedOnly`|
|     DLLs, environment           |   /`StoreSignedOnly` |
|     variables if needed_        |   process mitigations? |
|                                 | [[8]] |
+---------------------------------+------+
|  Verify validity of DLL<br />   | |
|     _i.e. check expected        | |
|     signature_                  | |
+--+--+

<br /><br />

### Nevertheless: the problem remains!{.emphasis}

# Detecting DLL Hijacking
## It's not straightforward...

A few (flawed) ideas:

- Look for known DLL hijack targets (DLL names, executables) - see [[9]]
- Look for creation of DLLs by unexpected processes
- Look for common targets (e.g. Microsoft-signed executables) in unexpected locations
- Look for common targets loading DLLs not on VT
- ...

![](img/sigma.png){.split}

# Detecting DLL Hijacking (2)
## ... but not impossible
* 🔎 Instead of just looking for the DLL Hijacking, look for the **behaviour that follows**

* 📚 **Layer your defences**!

* 🛡️ Ensure a **broad base** of behavioural rules, hunting techniques, anomaly detection

    * You're never going to detect everything
    * The more you monitor, the less likely an intrusion will go undetected


[1]: https://kevinalmansa.github.io/application%20security/DLL-Proxying/
[2]: https://www.trendmicro.com/vinfo/de/threat-encyclopedia/web-attack/54/stuxnet-malware-targets-scada-systems
[3]: https://wikileaks.org/ciav7p1/cms/page_27492385.html
[4]: https://wikileaks.org/ciav7p1/cms/page_3375327.html
[5]: https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/rpt-dll-sideloading.pdf
[6]: https://www.pwc.co.uk/issues/cyber-security-services/cyber-threat-intelligence.html
[7]: https://wietze.github.io/blog/hijacking-dlls-in-windows
[8]: https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-updateprocthreadattribute
[9]: https://github.com/wietze/windows-dll-hijacking/blob/master/possible_windows_dll_hijacking.yml
