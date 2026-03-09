<p align="center">
<img src="https://i.imgur.com/CtGfsq8.png" alt="osTicket logo"/>
</p>

<h1>Learning DNS Records & Caching in Azure</h1>

In this lab, we will experiment with **DNS** using the **Active Directory Lab**. This lab will help us have a better understanding of DNS.

The lab will demonstrate how to configure and test the **DNS Records** inside the **Active Directory Domain** hosted on Microsoft Azure, with the 2 VMs of

- **DC-1** → Windows Server Domain Controller **("mydomain.com")**
- **Client-1** → Windows 10 Pro Domain-joined client PC

(The DNS Server can be Integrated with Active Directory & automatically get installed with ADDS on the Windows Server.)
<br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- **Windows Server** (**Domain Controller** & **DNS Server**)
- **DNS Manager | DNS Management Console**
- PowerShell
- Command-line Tools | **ping, nslookup, ipconfig** 
<br />

<h2>💻 Operating Systems Used</h2>

- **Windows Server (22H2)**
- **Windows 10 Pro (22H2)**

<h2>List of Prerequisites</h2>

- **Active Directory setup (Client & Windows Server)**
- **Client Machine joined to the Domain**
<br />

<h3>🔵 A-Record exercise</h3>
<p>
  
  1. Connect/log into DC-1 as your Domain Admin account, `mydomain.com\jane_admin`

  2. Connect/log into Client-1 as an admin `mydomain\jane_admin`
  
  3. From Client-1, try to **ping** “mainframe” (Random Name) from PowerShell. Notice that it fails.

  4. Use `nslookup mainframe`, notice that it fails (no DNS record).
</p>
<p>
  <img src="https://i.imgur.com/hEK8bjk.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>

  When the computer tries to interact with any host’s name on the Network, whether it's browsing to it or trying to ping it, the computer will first check its **Local DNS cache** **(*Stored in Memory*)** for Speed,
  
  - If it’s not found, then it will check its **Local Host File**, 
  
  - If it’s not found there, then it will check its **DNS Server** as a **last resort**, also the slowest.

  Therefore, it appears that the name “mainframe” is not found in any of these three.  
</p>
<p>

  5. Go to 🔹️ **DNS Manager** on DC-1, and create a DNS A-record on DC-1 for “mainframe” and have it point to DC-1’s Private IP address (10.0.0.4).
</p>
<p>
  <img src="https://i.imgur.com/Kx9k6Z0.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
  <img src="https://i.imgur.com/xkjOzEw.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
  <img src="https://i.imgur.com/bCHACop.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  6. Go back to Client-1 and try to ping it. Observe that it works.
</p>
<p>
  <img src="https://i.imgur.com/xiG9ab2.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>

  An **A-Record** (**Address Record**) in DNS (Domain Name System) is a type of DNS record that maps a Domain Name to an IPv4 address.

  💡 📝 Notes:
  
  - A-Records only support 32-bit IPv4 addresses.
  - For **IPv6**, there's a similar record called an **AAAA Record**.
  - A Domain can have multiple A-Records pointing to different IPs for load balancing or redundancy.
</p>
<br />

<h3>🟢 Local DNS Cache exercise</h3>
<p>
  7. Go back to DC-1 and change the "mainframe's" A-Record to the IP address (8.8.8.8).
</p>
<p>
  <img src="https://i.imgur.com/pjvhP6v.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  
  8. Go back to Client-1 and **ping** “mainframe” again. Observe that it still pings the old IP address.
</p>
<p>
  <img src="https://i.imgur.com/Tlf0WAL.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
  Mainframe’s IP Address still points to 10.0.0.4.
</p>
<p>
  
  9. Observe the local DNS cache with the command, `ipconfig /displaydns`
</p>
<p>
  <img src="https://i.imgur.com/8pFkbxV.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  The A (Host) Record still shows the IP address 10.0.0.4; that’s a problem, as it should be 8.8.8.8 after the changes we made to the DNS server.

  Here's how we troubleshoot it❗
</p>
<p>
  
  10. Flush the DNS cache with the command `ipconfig /flushdns` to refresh or update.

  - To flush the DNS cache, we need to use PowerShell as an Admin.
</p>
<p>
  <img src="https://i.imgur.com/jsGO0le.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  
  11. Observe that the cache is empty with the command `ipconfig /displaydns`.
</p>
<p>
  <img src="https://i.imgur.com/RDlrYyJ.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  12. Attempt to ping “mainframe” again. Observe that the address of the new record is showing up, which is 8.8.8.8
</p>
<p>
  <img src="https://i.imgur.com/S0y2vDj.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  
  This shows why the `ipconfig /flushdns` command is good to use. If someone in the organization can’t access the same resources as everyone else, it may have something to do with the DNS and Local Cache. 
  
  It would be best to try refreshing or updating the DNS cache for troubleshooting ✅️. 
</p>
<h4>🔹️ Summarize</h4>
<p>
  The local DNS cache on a DNS server (or Client) is used to temporarily store DNS query results so that future requests for the same domain can be resolved more quickly without querying external DNS servers again.

  A DNS server's local cache can contain/store:
  | Record Type | Purpose                            |
  | ----------- | ---------------------------------- |
  | A           | IPv4 address for a domain          |
  | AAAA        | IPv6 address for a domain          |
  | CNAME       | Canonical name (alias) mapping     |
  | SRV         | Service records (used by AD, VoIP) |
  | MX          | Mail exchange servers              |
  | NS          | Name server for a domain           |

  <br />
  🔄 Example Flow:

  1. A user accesses "example.com".
  2. The DNS server queries the authoritative server and caches:
      - "example.com", A, 93.184.216.34, TTL = 3600
  3. Another user asks for "example.com" within the hour.
  4. DNS server responds from its local cache instantly✅️.
  <br />
  🧹 Cache Maintenance:

  - Cached entries expire after **TTL** (Time to Live).
  - Admins (and Users) can manually flush the cache if needed (e.g., after DNS changes):
      - `ipconfig /flushdns`          - Windows (client)
      - `Clear-DnsServerCache`        - PowerShell (DNS Server)
      - `rndc flush`                  - BIND (Linux)
  <br />
  ⚠️ In Active Directory Environments
  
  In AD-integrated DNS servers:

  - The local cache helps reduce repetitive lookups for internal resources like DCs, LDAP, and Kerberos.
  - It complements the zone data, which is the authoritative info for the domain.
</p>
<br />

<h3>🟦 CNAME Record exercise</h3>
<p>

  13. Go back to DC-1 and create a CNAME record that points the host “**search**” to “**www.google.com**”.
</p>
<p>
 <img src="https://i.imgur.com/gIB8cEA.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
 <img src="https://i.imgur.com/cCWXxSI.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  
  14. Go back to Client-1 and attempt to **ping** “search”, and observe the results of the CNAME record. 
</p>
<p>
  <img src="https://i.imgur.com/xRhK54m.png" height="90%" width="100%" alt="Disk Sanitization Steps"/>
</p>
<p>
  
  15. On Client-1, use the command `nslookup` “search”, and observe the results of the CNAME record.
</p>
<p>
  <img src="https://i.imgur.com/2At9LqL.png" height="90%" width="90%" alt="Disk Sanitization Steps"/>
</p>
<h4>🔹️ Summarize</h4>
<p>
  
  A **CNAME** record (**Canonical Name record**) in a DNS server is used to alias one domain name to another. In simpler terms, it tells DNS resolvers:

  - “This domain is just another name for that other domain.” or "**1 Human readable name to another Human readable name."**
  
  ✅ 1. Alias One Domain to Another
  
  It allows you to point multiple domain names to a single canonical (true) domain name.

  ✅ 2. Simplify DNS Management

  If you change the IP address of the canonical domain, you only need to update the A-record of that domain. All CNAMEs pointing to it will automatically follow the change — no need to update multiple A-records.
  
  <br />
  
  ❗ **Important Rules & Limitations:**

  | ⚠️ Rule / Limitation                        | Explanation                                                                  |
| ------------------------------------------- | ---------------------------------------------------------------------------- |
| ❌ CNAME must always point to another domain | It **cannot** point directly to an IP address (use A/AAAA records for that)  |
| ❌ No other records allowed on same name     | If `www.example.com` has a CNAME, it **cannot** have an A, MX, or TXT record |
| ✅ Multiple CNAMEs can point to one target   | Several domains can alias to a single canonical domain                       |

<br />
🔄 How CNAME Resolution Works:

  1. A client requests www.example.com.
  2. DNS sees a CNAME: www.example.com → "example.com".
  3. DNS then resolves "example.com" (usually via its A-record) to an IP.
  4. Client is sent the final actual IP.
<br />
🏢 In an Active Directory Environment

CNAME records are not usually used for core AD services (like domain controllers), because:

  - AD services (e.g., Kerberos, LDAP) often require A-records and SRV records.
  - Microsoft recommends using A-records for domain controllers and service records for reliability and compatibility.

However, CNAMEs might still be used for:

  - Aliasing internal tools or dashboards
  - Non-AD-related web services hosted internally or externally
</p>
<br />

---

## 🚀 What to Learn from DNS 
- How to create and test **A-Records** and **CNAME records** in Active Directory DNS.  
- How **Local DNS cache** can retain outdated records and how to flush it.  
- Hands-on troubleshooting with **ping, nslookup, and ipconfig**.  
- Reinforced real-world system administration tasks that are critical in enterprise IT.  

---
<h3>Conclusion</h3>
<p>
  This lab provided an overview of DNS (Domain Name System) using Active Directory Setup.
</p>

