# Xamarin
## Traffic Interception
### Resources
- [Intercepting Xamarin Mobile App Traffic (triskelelabs.com)](https://www.triskelelabs.com/blog/intercepting-xamarin-mobile-app-traffic-2#:~:text=For%20an%20attacker%20to%20intercept,settings%20to%20use%20this%20proxy.) **(Recommended)**
- [Capturing HTTP Requests from a non-proxy-aware Mobile Application | bhavukjain1](https://bhavukjain.com/blog/2023/02/19/capturing-requests-non-proxy-aware-application)
- [SSL Pinning Bypass for Android & iPhone Users | Appknox](https://www.appknox.com/blog/bypass-ssl-pinning-in-ios-app)
The following is outdated, but I opted to keep it in the notes for reference:
- [How To Capture Non-Proxy Aware Mobile Application Traffic (IOS & Android) Xamarin/Flutter -Pentesting | by salman syed | Medium](https://slmnsd552.medium.com/how-to-capture-non-proxy-aware-mobile-application-traffic-ios-android-xamarin-flutter-924fe044facf)
### Steps
Follow the steps in the [How To Capture Non-Proxy Aware Mobile Application Traffic (IOS & Android) Xamarin/Flutter -Pentesting | by salman syed | Medium](https://slmnsd552.medium.com/how-to-capture-non-proxy-aware-mobile-application-traffic-ios-android-xamarin-flutter-924fe044facf) blog to set up OpenVPN.
1. Delete all `iptable` rules, refer to [iptables(8) - Linux man page (die.net)](https://linux.die.net/man/8/iptables) for full context.
```bash
# Flush filter rules i.e: FOWARD, INPUT, OUTPUT
iptables -F

# Get NAT rule number
sudo iptables -t nat -v -L -n --line-number

# Flush nat rule i.e: PREROUTING, POSTROUTING
sudo iptables -t nat -D PREROUTING <rule_number>

# Allow all inbound traffic
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
```
2. Route traffic from your VPN interface and redirect to your host (Burp Suite)
```bash
# To forward to local port 8888 
iptables -t nat -A PREROUTING -i tun0 -p tcp --dport 443 -j REDIRECT --to-port 8888

# or to forward to a remtoe host
iptables -t nat -A PREROUTING -p tcp --dport 443 -s <mobile_ip> -j DNAT --to-destination <burp_host_ip>:<burp_listenting_port>

iptables -t nat -A POSTROUTING -p tcp --dport <burp_listenting_port> -d <burp_host_ip> -j SNAT --to-source <burp_host_ip>
```
3. Enable IP Forwarding
```bash
# Enable
sysctl -w net.ipv4.ip_forward=1

# Validate it works
cat /proc/sys/net/ipv4/ip_forward
```
> If you don't see traffic in Burp, checkout Burp's Dashboard - sometimes it's an SSL pinning issue.
> ![](/Screenshots/Pasted%20image%2020230803164141.png)


### Bypass SSL pinning
- Login to the application without intercepting the traffic
- Download Burp Certificate by navigating to `http://burp:8080`. 
- Find the Bundle path on the device using objection
	![](/Screenshots/Pasted%20image%2020230804155041.png)
- Find the certificate using `find` and `grep`
	```bash
	find . | grep -E "cer|der"
	```
	![](/Screenshots/Pasted%20image%2020230804155700.png)
- Replace the certificate with Burp's certificate. You may need to convert `der` certificate format to `.cer`. To do so, follow this blog: [Install Burpsuite’s or any CA certificate to system store in Android 10,11 and Kali linux. | by n00🔑 | Medium](https://pswalia2u.medium.com/install-burpsuites-or-any-ca-certificate-to-system-store-in-android-10-and-11-38e508a5541a)