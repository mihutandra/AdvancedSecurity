These are the solutions for all 10 exercises in Seminar 2.

1. **List hosts that respond to ping**

   ```bash
   nmap -sn -n -oG - 10.200.99.0/24 |
     grep 'Status: Up' |
     cut -d ' ' -f 2 > live-hosts.lst
   ```

2. **Make `wget` print `Welcome!` on every request**

   In one terminal:

   ```bash
   while true; do
     printf 'HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\nContent-Length: 8\r\nConnection: close\r\n\r\nWelcome!' |
       nc -l -p 1234
   done
   ```

   In another terminal:

   ```bash
   wget -qO- http://localhost:1234/
   ```

3. **Transfer `wget` between two `nc` instances**

   On the receiving VM:

   ```bash
   nc -l 9000 > wget.copy
   ```

   On the sending VM, replace `<RECEIVER_IP>` with the receiver’s lab IP:

   ```bash
   nc -N <RECEIVER_IP> 9000 < "$(command -v wget)"
   ```

   Then check the copy on the receiver:

   ```bash
   cmp "$(command -v wget)" wget.copy
   chmod +x wget.copy
   ./wget.copy --version
   ```

4. **Capture DHCP packets and a TCP handshake in Wireshark**

   We start a capture on the VM’s active interface. For DHCP, we use the display filter `dhcp` (older Wireshark versions may use `bootp`). In the isolated lab, we trigger a lease exchange, for example:

   ```bash
   sudo ipconfig set en0 DHCP
   ```

   We capture the DHCP exchange and take the screenshot. Releasing a lease can briefly interrupt that VM’s network.

   

   For the handshake, we generate a TCP connection to a reachable lab service, then use `tcp.flags.syn == 1` to locate the opening SYN and follow that packet’s stream. The stream shows SYN, SYN/ACK, and ACK together.

5. **Show cookies in HTTP GET requests; handle HTTPS**
    
    Replace `<CLIENT_IP>` with your VM’s IP:

   ```bash
   sudo tshark -i <IFACE> \
     -Y 'ip.src == <CLIENT_IP> && http.request.method == "GET" && http.cookie' \
     -T fields -e http.request.full_uri -e http.cookie
   ```
   
   Plain HTTP exposes cookies to packet capture. HTTPS encrypts them; to inspect HTTPS, you need to decrypt the session using authorized TLS session keys (for example, a browser key log configured in Wireshark) or capture through a trusted debugging proxy. Without decryption, the cookie header is not visible.

6. **Explain the HTTP POST filter**

   ```text
   tcp dst port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354)
   ```

   It selects TCP packets going to port 80. The high four bits of TCP byte 12 give the TCP header length in 32-bit words; shifting right by two converts that to bytes. The filter reads four bytes immediately after the TCP header—the start of the TCP payload—and compares them with `0x504f5354`, ASCII `POST`. This matches packets where the payload begins with `POST`, assuming the request line is in that packet.

7. **Create, list, and remove `test1` over FTP**

   **Using `nc`:** connect to the control port:

   ```bash
   nc <TARGET> 21
   ```

   We enter these FTP commands, waiting for each server reply:

   ```text
   USER service
   PASS service
   MKD test1
   PASV
   ```

   The server’s `227` reply gives a passive data address and port as six numbers: `h1,h2,h3,h4,p1,p2`. The data port is `p1*256+p2`. In a second terminal, we connect to that port with `nc <TARGET> <DATA_PORT>`, then send `LIST` in the original control connection. The listing appears in the data connection. Finally, in the control connection:

   ```text
   RMD test1
   QUIT
   ```

   **Using the `ftp` client:**

   ```bash
   ftp <TARGET> 21
   ```

   We log in as `service` with password `service`, then enter:

   ```text
   mkdir test1
   ls
   rmdir test1
   quit
   ```

8. **List open TCP ports and detected service versions**

   In the lab, we scan all TCP ports and save the report:

   ```bash
   nmap -sV -p- -T4 <TARGET> -oN scan.txt
   ```

   We extract only open ports, service names, and version text:

   ```bash
   awk '/^[0-9]+\/tcp[[:space:]]+open/ {
     port=$1; service=$3
     $1=$2=$3=""
     sub(/^[[:space:]]+/, "")
     print port, service, $0
   }' scan.txt
   ```

9. **Compare TCP and UDP scans**

   We capture traffic on the interface that reaches the target:

   ```bash
   sudo tcpdump -ni <IFACE> 'host <TARGET> and (tcp or udp or icmp)'
   ```

   In another terminal, we run the scans:

   ```bash
   sudo nmap -sS -p 21,53,80,1999 <TARGET>
   sudo nmap -sU -p 21,53,80,1999 <TARGET>
   ```

   A TCP SYN scan sends SYN probes. Open ports generally reply SYN/ACK; closed ports reply RST. Nmap’s SYN scan does not complete the TCP handshake. UDP has no handshake: a closed port often replies with ICMP port-unreachable, while an open service may reply at the application level or remain silent. Silence can leave a port classified as `open|filtered`.

   Blocking ICMP can hide UDP “closed” replies and make UDP results less certain. It can also interfere with Nmap’s host discovery, so `-Pn` tells Nmap to skip that discovery step. TCP port responses such as SYN/ACK and RST still provide information unless the firewall blocks those too.

10. **PHP and JavaScript shell scripts**

   To check connectivity without providing a shell, we use a fixed health-check request instead. For example, browser JavaScript can request an HTTPS endpoint:

   ```javascript
   fetch("https://10.0.0.20/health")
     .then(response => console.log(response.status))
     .catch(error => console.error(error));
   ```