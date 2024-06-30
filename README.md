# X-Via Scripts

Este repositório contém os scripts de configuração do X-Via utilizando [Ansible](https://www.ansible.com/).

## Ambientes

Os ambientes de desenvolvimento e homologação podem ser configurados através dos seguintes comandos:

```
# dentro do diretório xvia 
ansible-playbook -i hosts/dev/[dev|qa] xvia_[dev|qa].yml
```

Mais informações em [X-Via instruções de instalação](/xvia/README.md)

# AC - Autoridade Certificadora para testes

Os scripts incluem a configuração de uma autoridade certificadora para testes, mais informações em [AC de teste com Carimbo ACT e OCSP](/xvia/TESTCA.md)

**A Autoridade Certificadora de Testes não deve ser utilizada em produção**. A mesma instância pode ser configurada tanto em DEV quando em HOMOLOGA para evitar que novos certificados precisem ser gerados ao migrar servidores entre estes ambientes.

Uma [Autoridade Certificadora](https://www.iti.gov.br/icp-brasil/estrutura) certificada ICP-Brasil deve ser utilizada para o ambiente de produção.