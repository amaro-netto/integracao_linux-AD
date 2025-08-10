# 🎯 OBJETIVO
Fazer com que um computador Linux (Debian/Ubuntu) aceite login de usuários do Active Directory, como se fossem usuários locais.

Você precisa apenas:

**- Ter o nome do domínio AD (ex: empresa.local)**

**- Ter o IP ou nome do servidor AD**

**- Ter um usuário administrador do domínio para conectar**

##

### 🧩 ETAPA 1 – Preparar o sistema

✅ Passo 1.1: Verifique o nome do seu computador
Ele precisa ter um nome identificável pela rede.

```bash
hostnamectl set-hostname maquina01
```

✅ Passo 1.2: Sincronize o horário

O Active Directory exige horário sincronizado, ou a autenticação vai falhar.

```bash
sudo apt update
sudo apt install chrony -y
```
Edite o arquivo do chrony:

```bash
sudo nano /etc/chrony/chrony.conf
```

Adicione seu servidor AD (exemplo com IP 192.168.1.10):

nginx
Copiar
Editar
server 192.168.1.10 iburst
Salve e reinicie:

bash
Copiar
Editar
sudo systemctl restart chronyd
🧩 ETAPA 2 – Instalar os pacotes necessários
bash
Copiar
Editar
sudo apt update
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli oddjob oddjob-mkhomedir samba-common-bin krb5-user packagekit -y
Durante a instalação, pode aparecer a pergunta sobre o REALM do Kerberos. Coloque seu domínio em MAIÚSCULAS.
Exemplo:

pgsql
Copiar
Editar
EXEMPLO.LOCAL
🧩 ETAPA 3 – Descobrir o domínio Active Directory
Use o comando abaixo com o nome do seu domínio (tudo em maiúsculas ou minúsculas, tanto faz):

bash
Copiar
Editar
realm discover exemplo.local
Você deve ver uma saída parecida com:

yaml
Copiar
Editar
exemplo.local
  type: kerberos
  realm-name: EXEMPLO.LOCAL
  domain-name: exemplo.local
  configured: no
  ...
🧩 ETAPA 4 – Entrar no domínio
Substitua abaixo:

Administrador: usuário com permissão no domínio

exemplo.local: seu domínio

bash
Copiar
Editar
sudo realm join --user=Administrador exemplo.local
Vai pedir a senha do usuário.

Se tudo der certo, você verá nenhum erro e o Linux já estará autenticado.

🧩 ETAPA 5 – Testar se o domínio está funcionando
bash
Copiar
Editar
realm list
Você verá um bloco com dados do domínio conectado.

Agora teste buscar um usuário:

bash
Copiar
Editar
id usuario@exemplo.local
Substitua usuario por um usuário real do AD.
Se aparecer um monte de informações como UID e grupos: ✅ Sucesso!

🧩 ETAPA 6 – Permitir apenas certos usuários (opcional)
Por padrão, qualquer usuário do AD pode logar. Para restringir:

bash
Copiar
Editar
sudo realm permit --groups "TI"
Apenas usuários do grupo TI (no AD) poderão logar no Linux.

🧩 ETAPA 7 – Criar pastas home automaticamente
Sem isso, o usuário do AD loga, mas não tem uma pasta pessoal.

Ative com:

bash
Copiar
Editar
sudo pam-auth-update --enable mkhomedir
🧩 ETAPA 8 – Melhorar nomes dos usuários
Por padrão, você precisa logar como:

sql
Copiar
Editar
usuario@exemplo.local
Se quiser permitir login só com usuario, edite:

bash
Copiar
Editar
sudo nano /etc/sssd/sssd.conf
Altere/adicione estas linhas:

ini
Copiar
Editar
use_fully_qualified_names = False
fallback_homedir = /home/%u
Salve, corrija permissões e reinicie:

bash
Copiar
Editar
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
🧪 TESTE FINAL
Deslogue e, na tela de login do Linux, tente entrar com um usuário do AD:

Usuário: usuario

Senha: (senha do AD)

✅ Se logar e for levado à área de trabalho com uma pasta /home/usuario, tudo está funcionando perfeitamente!

🔐 Extras de segurança (opcional)
Bloquear logins fora do domínio
Só permitir logins via AD:

bash
Copiar
Editar
sudo realm deny --all
sudo realm permit --groups TI
🧰 DICA FINAL – Diagnóstico se algo der errado
bash
Copiar
Editar
journalctl -xe | grep sssd
realm discover exemplo.local
realm join --verbose --user=Administrador exemplo.local
