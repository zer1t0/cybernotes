# SCCM

## Terminology

- Site: A SCCM administrative environment, similar to the concept of domain in
  AD (or tenant in Azure or zone in DNS).

- [Central Administrative Site](https://learn.microsoft.com/en-us/intune/configmgr/core/plan-design/hierarchy/design-a-hierarchy-of-sites#BKMK_ChooseCAS) (CAS): In big environment a CAS can be used
  to manage several Primary Sites. Is the top site that manages the child
  primary sites (seems like the forest in AD).

- Primary Site Server: The main SCCM server that rules the site. It can have
  other server roles aswell like Distribution Point, Management Point, Site
  Database, etc.

- Distribution Point: A server that distributes software programs, updates or OS
  images (used by PXE).

- Management Point: A server that distributes policies and configurations.

- Site Database Server: A server that contains the database used by the site
  server to manage the clients.

## Resources

- [The Hacker Recipes: SCCM / MECM](https://www.thehacker.recipes/ad/movement/sccm-mecm/)
- [Misconfiguration-Manager](https://github.com/subat0mik/Misconfiguration-Manager/) by subat0mik, Mayyhem and [others](https://github.com/subat0mik/Misconfiguration-Manager/graphs/contributors)
- [Complete SCCM Installation Guide and Configuration](https://www.systemcenterdudes.com/complete-sccm-installation-guide-and-configuration/) by Benoit Lecours
- 2023 [Looting Microsoft Configuration Manager](https://rzec.se/blog/looting-microsoft-configuration-manager/) by Tomas Rzepka
- [sccmhunter (Tool)](https://github.com/garrettfoster13/sccmhunter) by garrettfoster13
- [SCCMSecrets (Tool)](https://github.com/synacktiv/SCCMSecrets) by q-roland
- [PXEThief (Tool)](https://github.com/MWR-CyberSec/PXEThief) by chrispanayi
- [pxethiefy (Tool)](https://github.com/csandker/pxethiefy) by csandker and Mayyhem
- [SharpSCCM (Tool)](https://github.com/Mayyhem/SharpSCCM) by Mayyhem
- [sccmhound (tool)](https://github.com/CrowdStrike/sccmhound)
