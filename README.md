# lets-encrypt
A library providing let's encrypt DNS providers for challenge resolution.

This project aims to automate renew certificates for the **Public** and **Private** 
WedSites, via [Let's Encrypt](https://letsencrypt.org/).  
Obtaining a certificate for private sites is due to the fact that the program solve the DNS-01 challenge.
It will allow to prove that you control the DNS for your domain name by putting a
specific value in a TXT record under that domain name. Then the DNS system will be queried 
by Let's Encrypt and if it finds a match, the certificate will be sent and save into files.


### Current Features
* Fully automated.
* Works well even if you have multiple web servers.
* Resolve ACME Lego Challenges.
  *  DNS (dns-01).
* Can talk to the Let's Encrypt CA.
* Create a Let's Encrypt account and save it.
* The private key is generated locally on your system.
* Free and Open Source Software, made with Go.


### Usage
#### Configuration file
```json
"lets_encrypt_user": {
    "mail": "example@gmail.com",
    "account_path": "/etc/letsencrypt/account"
},
"dns_servers": [
    {
      "name": "Name",
      "type": "pdns",
      "url": "http://0.0.0.0:8080",
      "api_key": "Api Key",
      "server_id": "localhost"
    }
],
"certificates_root_path": "/etc/ssl-alert-renew/letsencrypt/certificates",
```

If you want to create a configuration file, you can use [Viper](https://github.com/spf13/viper#putting-values-into-viper) to read,
and fill this structure by Unmarshalling the config file. The `mapstructure` will read all configuration file type.
```go
type Config struct {
    LetsEncryptUser lets_encrypt.LetsEncryptUserConfig `mapstructure:"lets_encrypt_user"`
    DNSServers      []dns.DNSServerConfig              `mapstructure:"dns_servers"`
    CertRootPath    string                             `mapstructure:"certificates_root_path"`
}
```
Use a custom DNS server to verify LE challenge and generate a new certificate

#### Without configuration file
```go
// Set location for both certificates and account directories
certificatesPath := "path/to/cert/root"
accountPath := "path/to/account"

// Initialize LE with a new user
leUser, _ := lets_encrypt.InitLetsEncryptUser(lets_encrypt.LetsEncryptUserConfig {
    Mail: "example@gmail.com",
    AccountPath: accountPath,
})
letsEncrypt, _ := lets_encrypt.InitLetsEncrypt(certificatesPath, leUser.GetLEUser())

// Use a custom powerDNS server as a provider for LE DNS challenge
dnsServer := dns.initDNSServer(dns.DNSServerConfig {
      Name: "Name",
      Type: "pdns",
      URL: "http://0.0.0.0:8080",
      APIKey: "Api Key",
      ServerID: "localhost"
})
letsEncrypt.SetDNSProvider(dns.DNSProvider{DNSServer: dnsServer})

// Retrieve a new certificate
letsEncrypt.AskCertificate("targeted.site.com")
```

When you will lunch the program Saved account/certificate example:
```
letsencrypt
    ├── account
    │   ├── privKey.pem
    │   ├── pubKey.pem
    │   └── registration.json
    └── certificates
        └── example.com.re
            ├── example.com.crt
            └── example.com.key

```
