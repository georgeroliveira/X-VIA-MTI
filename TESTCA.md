# AC de teste com Carimbo ACT e OCSP

**Note**. O AC de Teste deve ser utilizado _apenas para teste e desenvolvimento_. Não é uma Autoridade Certificadora segura.

O diretório `roles/xroad-ca/` contém uma coleção de scripts para configurar um ambiente de AC de teste baseado em openssl para assinar certificados e fornecer serviços TSA e OCSP durante o desenvolvimento.
Os scripts requerem o Ubuntu 18.04.

Você pode inicializar o servidor AC de teste automaticamente [com Ansible.](README.md). Isso inicializa um novo servidor de AC de teste se um estiver listado na categoria `ca-servers`.

Como alternativa, você pode inicializar o servidor manualmente. Isso é descrito nos capítulos 2-4. Se você usa o Ansible, pule esses capítulos.

Os capítulos 1 e 5-7 descrevem o uso do servidor AC inicializado e são úteis para instalação manual e responsável.

Para personalizar os detalhes do DN da AC de teste para a instalação Ansible, especifique os parâmetros em um arquivo em group_vars ([leia mais sobre group_vars](README.md))

Exemplo do arquivo de configuração group_vars com todos os parâmetros de AC de teste:

```
xroad_ca_o: "Customized Test"
xroad_ca_cn: "Customized Test CA CN"
xroad_ca_ocsp_o: "Customized Test"
xroad_ca_ocsp_cn: "Customized Test OCSP CN"
xroad_ca_tsa_o: "Customized Test"
xroad_ca_tsa_cn: "Customized Test TSA CN"
```

---------------------------------------------

## 1. Conteúdo do diretório roles/xroad-ca

* `bionic-files/lib/systemd/system` - systemd services para ACT e OCSP
* `common-files/etc/nginx` - nginx para proxy de requisicoes da ACT e OCSP
* `common-files/home/ca/CA` - Configuracoes da AC e scripts para assinar certificados localmente
* `common-files/home/ca/TSA` - A Autoridade de Carimbo de Tempo 
* `common-files/usr` - scripts para assinar certificados
* `templates/init.sh` - script para criar a autoridade certificadora de teste

---------------------------------------------

## 2. Preliminary requirements

(Ansible executa estes passos automaticamente)

1. Instalar o nginx-light na máquina de destino
2. Criar os seguintes usuários na máquina de destino
	- `ca`
	- `ocsp`
3. Copiar os seguintes diretórios de `roles/xroad-ca/[UBUNTU_RELEASE]-files` e `roles/xroad-ca/common-files` para a raíz da máquina de destino
	- `etc`
	- `home`
	- `usr`
  - `lib`
4. Copiar `roles/xroad-ca/templates/init.sh` para `home/ca/CA/`
5. Adicionar o usuário `ocsp` ao grupo `ca`
6. Dar permissão de propriedade ao usuário `ca` aos arquivos da pasta `/home/ca/CA`
7. Dar permissão de leitura ao grupo `ca` aos arquivos da pasta `/home/ca/CA`
8. Dar permissão de ler e executar ao grupo `ca` aos arquivos da pasta `/home/ca/CA`
9. Criar um arquivo chamado `ocsp.log` dentro da pasta `/var/log`
10. Permiss˙ao de propriedade ao usuário `ca` e ao grupo `ca` com leitura e escrita no arquivo `ocsp.log`
11. Preencher os parâmetros CA, OCSP and TSA distinguished names (DN) no arquivo `/home/ca/CA/init.sh`:
```
# dn parameters
DN_CA_O="{{ xroad_ca_o }}"
DN_CA_CN="{{ xroad_ca_cn }}"
DN_OCSP_O="{{ xroad_ca_ocsp_o }}"
DN_OCSP_CN="{{ xroad_ca_ocsp_cn }}"
DN_TSA_O="{{ xroad_ca_tsa_o }}"
DN_TSA_CN="{{ xroad_ca_tsa_cn }}"
```
Os valores dos campos campos CA, OCSP and TSA distinguished name podem ser definidos livremente,
a única limitação é que a combinação de valores deve ser única para cada três.

O uso de valores reconhecíveis e significativos ajuda a distinguir as autoridades de certificação ao registrá-las no servidor central.

---------------------------------------------

## 3. Criando uma nova Autoridade Certificadora

(O Ansible executa esta etapa automaticamente)

1. Como o usuário `ca`, execute `init.sh` dentro de `/home/ca/CA`

---------------------------------------------

## 4. (Re)iniciando o NGINX, AC, OCSP and ACT

(O Ansible executa esta etapa automaticamente)

1. Antes de iniciar os trabalhos, reinicie o serviço nginx para aplicar as alterações de proxy (sudo systemctl reload nginx)
2. Inicie inicie os serviços através do comando  `sudo systemctl start ca ocsp tsa`

---------------------------------------------

## 5. Sobre os servicos de AC, ACT e OCSP

Ambos os serviços usam o nginx como proxy para redirecionar as solicitações para as seguintes portas:

- POST requests na `porta 8888` são direcionadas para o processo em python iniciado pelo OCSP
- GET requests na url `/testca` e `porta 8888` são direcionadas para o processo em python iniciado pela AC
- requests (GET, POST) na `porta 8899` são direcionadas para `localhost:9999` no servidor em python iniciado pelo ACT

---------------------------------------------

## 6. Configurando o servidor central para uso com test-AC
Depois que os serviços foram iniciados com êxito, o test-AC está pronto para ser usado no ambiente de teste.

Para configurar o servidor central para usar o test-AC:

1. Importe os certificados CA, TSA e OCSP de `/home/ca/CA/certs` para o servidor central
    - `ee.ria.xroad.common.certificateprofile.impl.EjbcaCertificateProfileInfoProvider` pode ser usado como o provedor de informações de perfil de certificado.
2. Configure a AC para usar a autoridade certificadora de teste através da porta 8888 na máquina de teste-AC
3. Configure o ACT na porta 8899 na máquina de teste CA

---------------------------------------------

## 7. Assinando certificados

Para assinar um CSR, você tem três opções:

### 7.1. Use o comando `sign` em um arquivo

1. Faça o upload do CSR para o servidor de test-AC
2. Asine o certificado usando o comando `sign` (o usuário precisa ter direitos sudo)
```
user@some-ca-server:~$ sign sign_csr_20161020_member_FI_GOV_889.der
Using configuration from CA.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 6 (0x6)
        Validity
            Not Before: Oct 20 12:34:52 2016 GMT
            Not After : Oct 15 12:34:52 2036 GMT
        Subject:
            countryName               = FI
            organizationName          = FOONAME
            commonName                = 889
            serialNumber              = FI/cadev-ss1/GOV
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Key Usage: critical
                Non Repudiation
Certificate is to be certified until Oct 15 12:34:52 2036 GMT (7300 days)
Write out database with 1 new entries
Data Base Updated
```
O certificado assinado é armazenado na pasta `/home/ca/CA/newcerts/??.pem`, onde ?? = número de série

### 7.2. Use o comando `sign-sign` e `sign-auth` com piped data

Você pode usar SSH para assinar o CSR remotamente.
Escolha `sign-sign` ou` sign-auth` com base no tipo de certificado.

```
$ cat sign_csr_20161020_member_FI_GOV_901.der | ssh user@some-ca-server sign-sign > sign-cert-901.pem
Using configuration from CA.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4 (0x4)
        Validity
            Not Before: Oct 20 12:50:35 2016 GMT
            Not After : Oct 15 12:50:35 2036 GMT
        Subject:
            countryName               = FI
            organizationName          = 6
            commonName                = 901
            serialNumber              = FI/cadev-ss1/GOV
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            X509v3 Key Usage: critical
                Non Repudiation
Certificate is to be certified until Oct 15 12:50:35 2036 GMT (7300 days)
Write out database with 1 new entries
Data Base Updated
```
### 7.3 Use a interface web

Há uma interface da web simples sendo executada em http://some-ca-server:8888/testca/ que pode ser usada para assinar solicitações.

Também é possível usar um cliente http da linha de comando (por exemplo, curl):
```
curl -Fcertreq=@auth_csr.pem http://some-ca-server:8888/testca/sign
```

### 7.4 Certificate revocation

Para revogar um certificado assinado, você pode usar o script revoke.sh em `/home/ca/CA`.
Execute o script como um usuário com direitos sudo em `/home/ca/CA`.
evoke.sh usa como parâmetro o nome do arquivo do certificado assinado (.pem).

Exemplos:
```
$ cd /home/ca/CA/
$ ./revoke.sh /someplace/auth-cert-200.pem
Using configuration from CA.cnf
ERROR:Already revoked, serial number 04

$ ./revoke.sh /home/ca/CA/newcerts/05.pem
Using configuration from CA.cnf
Revoking Certificate 05.
Data Base Updated
```

## 8. Em caso de problemas

Os logs dos serviços Systemd podem ser visualizados com journalctl -u service-name, ex: `journalctl -u ocsp`.

