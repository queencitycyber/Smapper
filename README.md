<h1 align="center">
  <br>
  <a href="https://github.com/s0md3v/smap"><img src="/static/smap-logo.png" alt="Smap logo"></a>
</h1>

<h4 align="center">passive Nmap like scanner built with shodan.io</h4>


<p align="center"><img src="/static/smap-demo.png" alt="Smap demo"></p>

---

Smap is a replica of Nmap which uses shodan.io's free API for port scanning. It takes same command line arguments as Nmap and produces the same output which makes it a drop-in replacament for Nmap.  

This is just a simple fork. Instead of the top [1237 ports](https://api.shodan.io/shodan/ports) default ports scanned by Shodan and the original repo, I grabbed the [top 2000](https://www.shodan.io/search/facet?query=net%3A0%2F0&facet=port). 

To do this, I used a close combination of the following:


```bash
shodan stats --facets port:2000 net:0/0 > shodanclitop2k.txt
awk '{ print $1 }' shodanclitop2k.txt > shodantop2k.txt
(readarray -t ARRAY < shodantop2k.txt; IFS=','; echo "${ARRAY[*]}")
```

From there, it's just a simple swap.

## Features
- Scans 200 hosts per second
- Vulnerability detection
- Supports all nmap's output formats
- Service and version fingerprinting
- Makes no contact to the targets
- Doesn't require any account/api key

## Installation
```
go install -v github.com/queencitycyber/smap/cmd/smap@latest
```

## Usage
Smap takes the same arguments as Nmap but options other than `-p`, `-h`, `-o*`, `-iL` are ignored. If you are unfamiliar with Nmap, here's how to use Smap.

### Specifying targets
```
smap 127.0.0.1 127.0.0.2
```
You can also use a list of targets, seperated by newlines.
```
smap -iL targets.txt
```
**Supported formats**

```
1.1.1.1         // IPv4 address
example.com     // hostname
178.23.56.0/8   // CIDR
```

### Output
Smap supports 6 output formats which can be used with the `-o* ` as follows
```
smap example.com -oX output.xml
```
If you want to print the output to terminal, use hyphen (`-`) as filename.

**Supported formats**
```
oX    // nmap's xml format
oG    // nmap's greppable format
oN    // nmap's default format
oA    // output in all 3 formats above at once
oP    // IP:PORT pairs seperated by newlines
oS    // custom smap format
oJ    // json
```

> Note: Since Nmap doesn't scan/display vulnerabilities and tags, that data is not available in nmap's formats. Use `-oS` to view that info.

### Specifying ports
Smap scans these [2000 ports](https://raw.githubusercontent.com/queencitycyber/Smap/main/shodantop2k.md) by default. If you want to display results for certain ports, use the `-p` option.

```
smap -p21-30,80,443 -iL targets.txt
```

## Considerations
Since Smap simply fetches existent port data from shodan.io, it is super fast but there's more to it. You should use Smap if:

#### You want
- vulnerability detection
- a super fast port scanner
- results for most common ports (top 2000)
- no connections to be made to the targets

#### You are okay with
- not being able to scan IPv6 addresses
- results being up to 7 days old
- a few false negatives
