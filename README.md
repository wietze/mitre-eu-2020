# Hijacking DLLs in Windows
## Sixth EU MITRE ATT&CK® Community Workshop

This repository contains a talk on DLL Hijacking as presented to the EU MITRE ATT&CK® Community on 23 October 2020.

### Presentation
**Slides**: [HTML](https://wietze.github.io/eu-mitre-2020/) <img src="https://wietze.github.io/img/html.svg" alt="HTML" width="19"/> and [PDF](https://wietze.github.io/eu-mitre-2020/slides.pdf) <img src="https://wietze.github.io/img/pdf.svg" alt="PDF" width="19"/>

**Video**: this presentation has not been recorded.

### Used resources

**Original blog post**: [Hijacking DLLs in Windows](https://wietze.github.io/blog/hijacking-dlls-in-windows) (2020) via _wietze.github.io_

**Sigma rule**: [Possible Windows DLL Hijacking](https://github.com/wietze/windows-dll-hijacking/blob/master/possible_windows_dll_hijacking.yml) (2020) via _github.com_

**DLL Hijacking types**:
- DLL Proxying: [DLL Proxying](https://kevinalmansa.github.io/application%20security/DLL-Proxying/) (2017) via _kevinalmansa.github.io_
- DLL Replacement: [STUXNET Malware Targets SCADA Systems](https://www.trendmicro.com/vinfo/de/threat-encyclopedia/web-attack/54/stuxnet-malware-targets-scada-systems) (2010) via _trendmicro.com_
- DLL Search Order Hijacking: [Vault7: Chrome Portable DLL Hijack](https://wikileaks.org/ciav7p1/cms/page_27492385.html) (2017) via _wikileaks.org_
- DLL Phantom DLL Hijacking: [Vault7: Kaspersky "heapgrd" DLL Inject](https://wikileaks.org/ciav7p1/cms/page_3375327.html) (2017) via _wikileaks.org_
- WinSxS DLL replacement (aka DLL Side-loading): [A Thorn in the Side of the Anti-Virus Industry](https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/rpt-dll-sideloading.pdf) (2014) _via fireeye.com_
- Relative Path DLL Hijacking: [PwC Threat Intelligence](https://www.pwc.co.uk/issues/cyber-security-services/cyber-threat-intelligence.html) (2020) _via pwc.co.uk_

**Prevention**:
- Sigma rule: [Possible Windows DLL Hijacking](https://github.com/wietze/windows-dll-hijacking/blob/master/possible_windows_dll_hijacking.yml) (2020) via _github.com_
- PreferSystem32Images mitigation: [UpdateProcThreadAttribute function](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-updateprocthreadattribute) (2018) _via docs.microsoft.com_
