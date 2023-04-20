<h1>Splunk Lab</h1>

<h2>Infrastruktura:</h2>

Do twojej dyspozycji są dwie maszyny:

- splunk - maszyna z zainstalowanym splunkiem AIO (<i>all-in-one</i>)
- syslog_client - czysty Amazon Linux 2

<p>Do obu maszyn można dostać się przez SSH z uzyciem klucza prywatnego.</p>

```bash
ssh -i <private_key> ec2-user@<public_ip>
```

<p>Splunk ma również dostępne GUI na porcie 8000 po http.</p>
<p>Adresy IP, klucz prywatny oraz hasło do splunka zostaną tobie przekazane mailowo.</p>

<h2>Zadania:</h2>

Do wykonania są następujące zadania:

1. Zainstalować syslog-ng na maszynie syslog_client.
2. Stworzyć usera <i>splunk</i> na maszynie syslog_client.
3. Stworzyć plik konfiguracyjny dla syslog-ng, który pozwoli zapisywać logi z PA przychodzące na 514 (tcp/udp) do pliku.
4. Zainstalować Universal Forwarder na maszynie syslog_client.
5. Podpiąć UF do Deployment Servera znajdującego się na maszynie splunk.
6. Stworzyć index <i>firewall</i> na maszynie splunk.
7. Przygotować i wypchać konfigurację inputs.conf dla plików odbieranych przez syslog-ng.
8. Sprawdzić czy logi PA dostępne są w Splunku.
9. Przygotować konfigurację wyciągającą pole <i>hostname</i> z logów PA.
10. Usunąć przygotowaną konfigurację i zastąpić aplikacją <i>Palo Alto Networks Add-on for Splunk</i>

<h2>Przydatne materiały:</h2>

1. [Instalacja syslog-ng na Amazon Linux 2](https://www.syslog-ng.com/community/b/blog/posts/installing-syslog-ng-in-amazon-linux-2-including-graviton2)
2. [Instalacja Universal Forwardera](https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Installanixuniversalforwarder)
3. [Konfiguracja Deployment Servera](https://docs.splunk.com/Documentation/MSExchange/4.0.4/DeployMSX/Setupadeploymentserver)
4. [Podpięcie UF do DS](https://docs.splunk.com/Documentation/Forwarder/9.0.3/Forwarder/Configuretheuniversalforwarder)
5. [Monitorowanie pliku przy użyciu inputs.conf](https://docs.splunk.com/Documentation/Splunk/9.0.3/Data/Monitorfilesanddirectorieswithinputs.conf)
6. [Ekstrakcja pól przy użyciu props.conf](https://docs.splunk.com/Documentation/Splunk/9.0.3/Knowledge/Createandmaintainsearch-timefieldextractionsthroughconfigurationfiles)
7. [Aplikacja Palo Alto Networks Add-on for Splunk](https://splunkbase.splunk.com/app/2757)

<h2>Config do syslog-ng:</h2>

```
source network_syslog_udp {
  udp(port(514));
};

source network_syslog_tcp {
  tcp(port(514) max-connections(256) flags(no-parse));
};

destination network_syslog_dest {
  file("/opt/syslog/${SOURCEIP}/${HOST}_${YEAR}${MONTH}${DAY}_${HOUR}00.log"
  create-dirs(yes)
  perm(0600)
  dir-perm(0755)
  dir-owner(splunk)
  dir-group(splunk)
  owner(splunk)
  template("${MESSAGE}\n")
  group(splunk));
};

log {
  source(network_syslog_udp);
  source(network_syslog_tcp);
  destination(network_syslog_dest);
};
```

<h2>Komenda do pobrania UFa:</h2>

```bash
wget -O splunkforwarder-9.0.4-de405f4a7979-Linux-x86_64.tgz "https://download.splunk.com/products/universalforwarder/releases/9.0.4/linux/splunkforwarder-9.0.4-de405f4a7979-Linux-x86_64.tgz"
```