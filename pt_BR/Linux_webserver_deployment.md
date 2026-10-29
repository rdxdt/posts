> **Aviso:** Este artigo foi traduzido do inglês para o português brasileiro por uma inteligência artificial, como um experimento. Podem existir erros ou trechos com tradução imprecisa, em caso de dúvida consulte a [versão original em inglês](../en_US/Linux_webserver_deployment.md).

# Implantação de um servidor web GNU/Linux
Neste artigo eu vou abordar como implantar de forma razoavelmente segura um servidor web nginx em uma VPS linux, note que isto não vai incluir a configuração de um banco de dados, nem de PHP ou de outras aplicações wsgi.
Este será um servidor web básico, mas razoavelmente seguro, se você quer alugar uma VPS para fazer landing pages para clientes e hospedar vários sites no mesmo host, você vai conseguir economizar dinheiro hospedando vários sites estáticos na mesma máquina.
Note que todos os comandos serão precedidos por # ou $, # significa que você executa como root(ou com sudo) e $ como o seu usuário comum, mas estes são prompts, **não os copie como parte da linha de comando**.

## Provisionando a sua VPS
Existem vários provedores baratos de VPS na nuvem, se você encontrar um bom com um preço razoável não esqueça de procurar por cupons de desconto.
Eu posso listar alguns que eu conheço, mas eu não posso garantir a estabilidade ou os bons preços deles, se você quiser você também pode hospedar em casa, mas o investimento em ter um endereço ip público estático, não estar atrás de cgnat, um servidor, um nobreak para o seu servidor e para os equipamentos de rede, além do consumo de energia, simplesmente não vale a pena.
Também para este exemplo nós vamos usar o Debian, no momento em que escrevo estamos usando o Debian 13 mas se você está lendo isto no futuro use uma versão mais nova
Provedores populares:
- [Linode/Akamai Cloud](https://www.linode.com/lp/refer/?r=f0c57e62c8c9a44008452111b93a8f8fc6fa4028): Eles têm instâncias de CPU compartilhada a partir de $5 por mês e uma grande variedade de localizações físicas, com preços variados, pessoalmente eu já testei e uso em produção e tem sido perfeito.
- [Digital Ocean](https://www.digitalocean.com/): Nunca testei, mas eles oferecem instâncias de computação a partir de $4 por mês, mas é bem popular.
- [Vultr](https://www.vultr.com/): Eu já testei, e teve um bom desempenho, mas eu nunca usei em produção, cancelar os serviços deles não foi agradável.
- [Hetzner](https://www.hetzner.com/): Popular, e confiável, os sites de algumas distros GNU/Linux são hospedados nele.
- [Oracle Cloud](https://www.oracle.com/cloud/):Eles têm um plano gratuito, mas eu ouvi falar que eles são um pouco ruins, a interface de rede não é estável, e quando eu tentei usar a minha conta ficou presa em um limbo e ninguém conseguiu me ajudar, mas você pode testar o plano gratuito e tentar dar sorte.
AVISO: Tenha certeza de escolher uma senha de root forte!!!

Agora que você tem o seu servidor, e nós vamos usar 198.51.100.1 como o IPv4 dele e 2001:db8:1234:5678::1 como o IPv6 dele, é hora de acessá-lo por SSH.
`$ ssh root@198.51.100.1`
Agora o ssh deve pedir a sua senha de root, digite-a e você vai cair em um shell de root na sua vps.

## Atualizando o servidor
Agora que estamos dentro, a primeira coisa a fazer é ter certeza que estamos rodando um sistema atualizado, então execute o seguinte comando
`# apt update && apt full-upgrade -y`
Isto vai atualizar a lista de pacotes e vai atualizar todos os pacotes para a última versão, se houver atualizações do kernel reinicie o seu sistema agora com o comando `reboot` e conecte novamente quando a vps terminar de iniciar.

## Adicionando o seu usuário comum
A primeira coisa que precisamos fazer é criar o nosso usuário comum e adicioná-lo ao grupo sudoer, para este exemplo nós vamos usar o usuário `remote` para diferenciar, mas é conveniente usar o nome de usuário da sua estação de trabalho no seu servidor pois isso vai evitar que você tenha que digitar o seu nome de usuário toda vez que conectar no ssh, então no servidor digite o seguinte comando
`# adduser remote`
Agora ele vai pedir a sua senha, escolha uma boa que você consiga lembrar, tenha em mente que a mesma senha pode ser usada para escalar para root com o comando `sudo` então escolha uma forte.
Você pode deixar as outras informações vazias, mas agora precisamos adicionar o nosso novo usuário ao grupo sudoer com o seguinte comando:
`# usermod -aG sudo remote`
Este comando é o usermod, ele modifica uma característica de um usuário, o parâmetro -aG adiciona o nosso a um grupo, `sudo` é o nome do grupo e `remote` é o nome do usuário.

## Instalando os pacotes necessários
Agora nós vamos instalar todos os pacotes necessários para o nosso servidor básico, se você está planejando implantar por exemplo aplicações php+mysql/mariadb, você vai ter que adicionar os nomes dos pacotes delas aqui, ou você pode instalá-los depois, aliás eu vou usar o vim como editor de texto, ele pode ser desafiador para iniciantes, mas se você preferir usar o nano, fique à vontade para omitir o pacote `vim` no próximo comando.
Então execute o seguinte comando:
`# sudo apt install tmux vim nginx wireguard`
Agora espere a instalação dos pacotes terminar.

## (OPCIONAL) Gerando as suas chaves ssh
Este passo só é necessário se você não tiver um par de chaves ssh no seu **sistema local**, você vai digitar os seguintes comandos **como o seu usuário comum**:
`$ mkdir -p ~/.ssh && chmod 700 ~/.ssh && ssh-keygen`
Isto vai criar um diretório, torná-lo legível e gravável somente para o seu usuário como medida de segurança, e vai gerar o par de chaves.

## Copiando a sua chave pública para o servidor remoto
No seu **sistema local** execute o seguinte comando:
`$ ssh-copy-id remote@198.51.100.1`
Ele vai pedir a senha do seu usuário remoto, e então vai instalar a sua chave pública na home do seu usuário remoto, só lembre que `remote` é o nosso usuário de exemplo, troque pelo nome de usuário que você escolheu quando criou o usuário.

## Protegendo o servidor SSH
Agora é hora de uma medida de segurança crítica, como senhas são em sua maioria vulneráveis a força bruta, nós devemos usar autenticação por chave pública para que somente com a nossa chave alguém consiga entrar no sistema, também devemos desabilitar o login de root porque o usuário é fácil de adivinhar já que é o nome padrão do super usuário, e devemos desabilitar a autenticação por senha, então como **root** com o seu editor de texto favorito no seu servidor abra o arquivo `/etc/ssh/sshd_config` e faça as seguintes alterações:
- Localize a linha com `PermitRootLogin yes` e altere para `PermitRootLogin no`.
- Localize a linha com `#PubkeyAuthentication yes` e altere para `PubkeyAuthentication yes`.
- Localize a linha com `PasswordAuthentication yes` e altere para `PasswordAuthentication no`.
Salve o arquivo e saia do editor de texto.

Agora digite o comando `# systemctl restart sshd` e ele vai reiniciar o serviço ssh para que as alterações que você fez na configuração tenham efeito.


## Configure o seu túnel VPN Wireguard
A configuração do wireguard é bem fácil, primeiro você vai precisar gerar o par de chaves do servidor, então conectado ao servidor por ssh vire root com `$ sudo -i`, depois `# cd /etc/wireguard`, agora nós vamos gerar o par de chaves com este comando `# wg genkey | tee privatekey | wg pubkey > publickey`.
Também tenha certeza de ter um par de chaves wireguard na sua máquina(usando o mesmo comando acima, mas você poderia colocá-lo em `~/.wireguard`)
Agora crie um arquivo de configuração do wireguard no servidor chamado `wg0.conf` com o seu editor de texto favorito e cole o seguinte
```
[Interface]
Address = 10.0.0.1/32
ListenPort = 51821
PrivateKey = replace with your server private key

[Peer]
PublicKey = replace with your computer PUBLIC key
AllowedIPs = 10.0.0.2/32
```
Substitua `replace with your server private key` pela chave privada do seu servidor e `replace with your computer PUBLIC key` pela chave PÚBLICA do seu computador.
Tenha certeza de adicionar novos peers para cada máquina que você quer que consiga acessar este servidor pelo wireguard.
Depois disso execute os seguintes comandos para habilitar e iniciar a sua interface wireguard no servidor `# systemctl enable wg-quick@wg0 && systemctl start wg-quick@wg0.service`
Depois disso na sua máquina, habilite a sua conexão wireguard e tente pingar o ip wireguard do servidor(no caso deste exemplo é 10.0.0.1), se o ping for bem sucedido então você terminou de configurar o wireguard, se o ping falhar você deve resolver o problema antes de continuar.

## Configure o firewall
Agora que temos conectividade pelo wireguard, nós devemos conseguir configurar o firewall para deixar o ssh acessível somente pelo wireguard, isto vai fazer com que somente os seus sistemas que foram configurados como peers no wireguard acessem o ssh.
Então `# cd /etc` para mudar o seu diretório atual para o /etc e agora edite o arquivo `nftables.conf` com o seu editor de texto favorito e cole o seguinte conteúdo, também tenha certeza de editar o `eth0` para corresponder ao nome da interface de rede do seu servidor(verifique com o comando `$ ip link`)
```
#!/usr/sbin/nft -f

flush ruleset


table inet filter {
        chain input {
                type filter hook input priority 0; policy drop;
                 # accept any localhost traffic
                iif lo accept
                # accept previously allowed traffic
                ct state established,related accept
                ct state invalid log prefix "nftables CHAIN=input STATE=invalid " counter drop
                # accept ssh only over wg0(wireguard) interface
                iifname wg0 tcp dport 22 ct state new counter accept
                #accept http incoming connections
                iifname eth0 tcp dport 80 ct state new counter accept
                #accept https incoming connections
                iifname eth0 tcp dport 443 ct state new counter accept
                #accept wireguard incoming connections
                iifname eth0 udp dport 51821 ct state new counter accept
                #accept http incoming connections from vpn
                iifname wg0 tcp dport 80 ct state new counter accept
                #accept icmp requests
                ip protocol icmp accept
                ip6 nexthdr icmpv6 accept
                #add log prefix
                log prefix "nftables CHAIN=input POLICY=drop "
        }
        chain output {
                type filter hook output priority 0;
        }
}

```
Agora que você editou o arquivo vamos habilitar o firewall com o comando `# systemctl enable nftables && systemctl start nftables`
## Configure o nginx para sites estáticos
Agora que temos o nosso servidor protegido, nós vamos fazer a configuração do nosso site estático, para o nosso exemplo o nome do nosso site será `site` então execute `# cd /etc/nginx/sites-available/` e crie o seu arquivo de configuração com o nome do nosso site e cole o seguinte conteúdo, leia com atenção e substitua com as informações relevantes onde os comentários indicarem.
```
server {
    listen 80;
    listen [::]:80;

    # Replace with the real domain(s)
    server_name example.com www.example.com;

    # Replace with the path to your site rootdir(that is accesible for www-data)
    root /var/www/your-site-root-dir;
    index index.html;
    charset utf-8;

    # Don't advertise the nginx version
    server_tokens off;

    # Send relative Location headers so redirects keep the scheme/host/port
    # the client used (works unchanged once SSL is added in front)
    absolute_redirect off;

    # Static site: no uploads, no request bodies
    client_max_body_size 1k;

    # Never list directory contents
    autoindex off;

    # Security headers (inherited by every location that has no add_header of its own)
    add_header Content-Security-Policy "default-src 'none'; style-src 'self'; img-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=(), interest-cohort=()" always;
    add_header Cross-Origin-Opener-Policy "same-origin" always;

    # Only GET and HEAD make sense for a static site
    if ($request_method !~ ^(GET|HEAD)$) {
        return 405;
    }

    # Hide the .html extension: redirect requests that include it to the clean URL.
    # $request_uri is the original request, so nginx's internal rewrites
    # (index -> /index.html, try_files -> $uri.html) don't trigger these.
    if ($request_uri ~ "^/index(\.html)?(\?|$)") {
        return 301 /$is_args$args;
    }
    if ($request_uri ~ "^/(.+)\.html(\?|$)") {
        return 301 /$1$is_args$args;
    }

    # Custom error page, also used by the "return 404" rules below
    error_page 404 /404.html;

    location = /404.html {
        internal;
    }

    # Don't serve the error page with a 200 at its clean URL
    location = /404 {
        return 404;
    }

    # Home page: served by the index directive (index.html)
    location = / {
    }

    location / {
        # /articles -> articles.html. Directories are never tried ($uri/ is
        # left out), so subdirectory URLs like /images/ return 404.
        try_files $uri $uri.html =404;
    }

    # Block hidden files (.git, .env, ...) - the .well-known directory stays reachable
    location ~ /\.(?!well-known/) {
        return 404;
    }

    # Cache static assets in the browser (expires doesn't reset the add_header list)
    location ~* \.(css|svg|png|jpe?g|gif|webp|ico|woff2?)$ {
        try_files $uri =404;
        expires 7d;
        access_log off;
    }
}

```
Salve o arquivo e saia do editor de texto.

O nginx só carrega as configurações que estão dentro de `/etc/nginx/sites-enabled/`, então para habilitar o nosso site precisamos criar um link simbólico do nosso arquivo de configuração lá, execute o seguinte comando:
`# ln -s /etc/nginx/sites-available/site /etc/nginx/sites-enabled/site`
Este comando é o ln, ele cria links entre arquivos, o parâmetro -s faz dele um link simbólico, o primeiro caminho é o arquivo para o qual estamos apontando e o segundo é onde o link será criado, só lembre que `site` é o nosso nome de exemplo, troque pelo nome que você escolheu para o seu arquivo de configuração. Desta forma se você algum dia quiser desabilitar um site você só precisa remover o link simbólico de `sites-enabled` e o seu arquivo de configuração continuará seguro em `sites-available`.

Antes de aplicar as alterações nós devemos testar a configuração, um erro de digitação no arquivo de configuração pode fazer o nginx falhar ao iniciar e derrubar todos os seus sites, então execute o seguinte comando:
`# nginx -t`
Se estiver tudo certo você deve ver uma saída como esta:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Se o teste falhar, o nginx vai te dizer o arquivo e a linha onde está o erro, então corrija e execute `# nginx -t` novamente até o teste ser bem sucedido.

Agora que a configuração foi testada, recarregue o nginx para que ele comece a servir o seu site com o comando `# systemctl reload nginx`, o reload aplica a nova configuração sem derrubar as conexões dos sites que já estão sendo servidos.

## Configure a proteção do cloudflare
Agora que o nosso site está sendo servido, nós vamos colocá-lo atrás do Cloudflare. O Cloudflare vai agir como um proxy reverso entre os visitantes e o nosso servidor, então os visitantes só vão ver endereços IP do Cloudflare em vez do IP do nosso servidor, e ele também vai proteger o nosso servidor contra ataques DDoS e fazer cache dos nossos arquivos estáticos na CDN deles, e o plano gratuito é mais do que suficiente para sites estáticos.
Para este exemplo nós vamos continuar usando `example.com` como o nosso domínio, substitua pelo seu domínio real.

### Criando a sua conta no Cloudflare
Vá até o [painel do Cloudflare](https://dash.cloudflare.com/sign-up) e crie a sua conta com o seu email e uma senha forte, depois disso verifique o seu endereço de email com o link que eles vão te enviar.
AVISO: Quem tiver acesso a esta conta controla para onde o seu domínio aponta, então tenha certeza de habilitar a autenticação de dois fatores nas configurações do seu perfil!!!

### Adicionando o seu domínio ao Cloudflare
No painel clique para adicionar um domínio, digite o nome do seu domínio(somente `example.com`, sem `www` ou `https://`) e continue, agora escolha o plano **Free**.
O Cloudflare vai escanear os registros DNS que o seu domínio tem atualmente e importá-los, revise esta lista com atenção, principalmente se você tem email neste domínio, porque se os registros `MX` e `TXT` estiverem faltando o seu email vai parar de funcionar depois da migração.

### Configurando os registros DNS
Ainda na página de registros DNS, apague quaisquer registros `A`, `AAAA` ou `CNAME` que apontem para uma hospedagem antiga e adicione os seguintes registros apontando para o nosso servidor:

| Tipo | Nome | Conteúdo | Status do proxy |
|------|------|----------|-----------------|
| A | @ | 198.51.100.1 | Proxied |
| AAAA | @ | 2001:db8:1234:5678::1 | Proxied |
| CNAME | www | example.com | Proxied |

O `@` significa o próprio domínio(`example.com`) e o status **Proxied**(a nuvem laranja) é o que faz o tráfego passar pelo Cloudflare, se um registro estiver marcado como **DNS only**(a nuvem cinza) o Cloudflare vai simplesmente responder com o endereço IP real do seu servidor e você não terá nenhuma proteção.
Tenha em mente que o Cloudflare só faz proxy de tráfego HTTP e HTTPS, então não crie um registro como `vpn.example.com` para usar como o seu endpoint do wireguard, ele teria que ser **DNS only** e expor o endereço IP do seu servidor para qualquer um, continue usando o endereço IP diretamente na sua configuração do wireguard.

### Migrando o DNS para o Cloudflare
Depois de revisar os registros, o Cloudflare vai te mostrar dois nameservers, algo como `ada.ns.cloudflare.com` e `bob.ns.cloudflare.com`, anote-os porque cada conta recebe o seu próprio par.
Agora entre no seu registrador de domínio(a empresa onde você comprou o seu domínio), se o DNSSEC estiver habilitado para o seu domínio desabilite-o primeiro, senão o seu domínio vai parar de resolver depois da alteração, então encontre as configurações de nameservers do seu domínio, substitua os nameservers atuais pelos dois do Cloudflare e salve.
A alteração pode levar de alguns minutos até 24 horas para propagar, o Cloudflare vai te enviar um email quando o seu domínio estiver ativo, mas você também pode verificar no seu **sistema local** com o seguinte comando:
`$ dig NS example.com +short`
Quando a saída mostrar os nameservers do Cloudflare a migração está concluída, se você desabilitou o DNSSEC você pode habilitá-lo novamente no painel do Cloudflare em **DNS > Settings**, ele vai te dar um registro DS que você tem que adicionar no seu registrador.

### Testando o proxy do Cloudflare
Agora que o domínio está ativo, vamos ter certeza que o tráfego está passando pelo Cloudflare, então no seu **sistema local** execute o seguinte comando:
`$ curl -I http://example.com`
Nos cabeçalhos da resposta você deve ver `server: cloudflare` e um cabeçalho `cf-ray`, também `$ dig A example.com +short` deve retornar endereços IP do Cloudflare em vez de 198.51.100.1, se for este o caso o seu site agora está protegido pelo Cloudflare.
Como o nosso servidor só escuta na porta 80, o Cloudflare vai se conectar a ele por HTTP simples, os visitantes ainda vão conseguir acessar o seu site com HTTPS porque o Cloudflare fornece o certificado para eles, você pode verificar isso no painel em **SSL/TLS > Overview**, nós vamos criptografar a conexão entre o Cloudflare e o nosso servidor mais tarde na seção do certbot.

### (OPCIONAL) Permitindo somente os IPs do Cloudflare no firewall
Neste momento qualquer um que saiba o endereço IP do seu servidor ainda consegue acessar o site diretamente e pular o Cloudflare completamente, então nós vamos alterar o firewall para aceitar conexões HTTP e HTTPS somente dos endereços IP do Cloudflare.
A lista de endereços IP do Cloudflare é publicada em [https://www.cloudflare.com/ips-v4](https://www.cloudflare.com/ips-v4) e [https://www.cloudflare.com/ips-v6](https://www.cloudflare.com/ips-v6), a lista abaixo foi tirada de lá no momento em que escrevo, mas ela pode mudar, então compare com estes links antes de usá-la.
Como root edite o arquivo `/etc/nftables.conf` com o seu editor de texto favorito, adicione os dois sets logo depois da linha `table inet filter {`:
```
        set cloudflare_v4 {
                type ipv4_addr
                flags interval
                elements = {
                        173.245.48.0/20, 103.21.244.0/22, 103.22.200.0/22,
                        103.31.4.0/22, 141.101.64.0/18, 108.162.192.0/18,
                        190.93.240.0/20, 188.114.96.0/20, 197.234.240.0/22,
                        198.41.128.0/17, 162.158.0.0/15, 104.16.0.0/13,
                        104.24.0.0/14, 172.64.0.0/13, 131.0.72.0/22
                }
        }

        set cloudflare_v6 {
                type ipv6_addr
                flags interval
                elements = {
                        2400:cb00::/32, 2606:4700::/32, 2803:f800::/32,
                        2405:b500::/32, 2405:8100::/32, 2a06:98c0::/29,
                        2c0f:f248::/32
                }
        }
```
Agora substitua as duas regras que aceitam conexões de entrada http e https na `eth0` pelas seguintes regras:
```
                #accept http and https incoming connections only from cloudflare
                iifname eth0 ip saddr @cloudflare_v4 tcp dport { 80, 443 } ct state new counter accept
                iifname eth0 ip6 saddr @cloudflare_v6 tcp dport { 80, 443 } ct state new counter accept
```
Mantenha a regra que aceita http da `wg0`, desta forma você ainda consegue acessar o servidor diretamente pelo wireguard para resolver problemas do site.
Salve o arquivo e saia do editor de texto, antes de aplicar as alterações verifique se o arquivo tem erros com o seguinte comando:
`# nft -c -f /etc/nftables.conf`
O parâmetro -c só verifica o arquivo sem aplicá-lo, se não houver saída o arquivo está correto, então aplique-o com `# systemctl restart nftables`.
Agora tente acessar `http://198.51.100.1` no seu navegador com o wireguard desconectado, a conexão deve expirar, enquanto `http://example.com` deve continuar funcionando pelo Cloudflare.
Só lembre que se o Cloudflare adicionar novos endereços IP à lista deles e você não atualizar o seu firewall, os visitantes que chegarem a estes novos servidores do Cloudflare não vão conseguir acessar o seu site, então verifique a lista de tempos em tempos.



## Habilitando HTTPS com o certbot
Neste momento a conexão entre o visitante e o Cloudflare é criptografada, mas a conexão entre o Cloudflare e o nosso servidor é HTTP simples, então qualquer um no caminho entre eles pode ler ou modificar as nossas páginas. Para corrigir isso nós vamos obter um certificado gratuito do Let's Encrypt com o certbot e fazer o Cloudflare falar com o nosso servidor somente por HTTPS.
Então instale o certbot e o plugin nginx dele com o seguinte comando:
`# apt install certbot python3-certbot-nginx`
Agora nós vamos solicitar o certificado, substitua os domínios e o diretório raiz pelos do seu arquivo de configuração do nginx:
`# certbot --authenticator webroot --webroot-path /var/www/your-site-root-dir --installer nginx --redirect -d example.com -d www.example.com`
Este comando usa dois plugins, o autenticador `webroot` prova para o Let's Encrypt que você é dono do domínio colocando um arquivo temporário dentro do diretório raiz do seu site(em `.well-known/acme-challenge/`, é por isso que a nossa configuração não bloqueia o diretório `.well-known`), e o instalador `nginx` edita o seu arquivo de configuração do nginx para usar o certificado. Nós usamos o autenticador `webroot` em vez do `nginx` porque ele continua funcionando atrás do Cloudflare quando as renovações acontecem por HTTPS, o parâmetro `--redirect` faz o nginx redirecionar toda requisição HTTP para HTTPS e cada `-d` é um domínio que será incluído no certificado.
O certbot vai pedir o seu endereço de email, usado para te avisar se algo der errado com o seu certificado, e para aceitar os termos de serviço, depois disso ele vai emitir o certificado e recarregar o nginx.
Se você abrir o seu arquivo de configuração em `/etc/nginx/sites-available/site` você vai ver que o certbot adicionou o `listen 443 ssl` e as linhas do certificado ao nosso bloco server e moveu as linhas `listen 80` para um novo bloco server que só redireciona para HTTPS, todas estas linhas terminam com o comentário `# managed by Certbot`.

AVISO: Faça o próximo passo imediatamente, enquanto o Cloudflare ainda estiver se conectando por HTTP o seu site ficará preso em um loop de redirecionamento!!!

Agora no painel do Cloudflare vá em **SSL/TLS > Overview**, escolha o modo **Custom** e selecione **Full (strict)**, desta forma o Cloudflare sempre vai se conectar ao nosso servidor por HTTPS e vai se recusar a conectar se o nosso certificado não for válido.
Vamos testar o certificado diretamente no servidor, conectado por ssh execute o seguinte comando:
`$ curl -I --resolve example.com:443:127.0.0.1 https://example.com`
O parâmetro `--resolve` faz o curl se conectar ao nosso próprio servidor em vez do Cloudflare, e como o curl verifica o certificado, se você receber uma resposta com `HTTP/1.1 200 OK` o certificado está correto. Agora abra `https://example.com` no seu navegador e verifique se o site carrega normalmente.
Os certificados do Let's Encrypt só são válidos por 90 dias, mas o pacote do certbot já instala um timer do systemd que os renova automaticamente, para ter certeza que a renovação vai funcionar execute o seguinte comando:
`# certbot renew --dry-run`
Se ele disser que a renovação simulada foi bem sucedida você não precisa mais se preocupar com o seu certificado.

## Registrando os IPs reais dos visitantes
Como todo o tráfego agora passa pelo Cloudflare, os logs do nginx só vão mostrar endereços IP do Cloudflare como os visitantes, o que torna os logs inúteis para descobrir quem está acessando o seu site. O Cloudflare envia o endereço IP real do visitante no cabeçalho `CF-Connecting-IP`, então nós vamos dizer ao nginx para confiar neste cabeçalho, mas somente quando a requisição vier de um endereço IP do Cloudflare, senão qualquer um poderia falsificar o seu IP enviando este cabeçalho diretamente para o nosso servidor.
Como root crie o arquivo `/etc/nginx/conf.d/cloudflare.conf` com o seu editor de texto favorito e cole o seguinte conteúdo(as faixas de IP são as mesmas da seção do firewall, então compare-as com as listas do Cloudflare antes de usá-las):
```
# Cloudflare IPv4 ranges
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
set_real_ip_from 103.31.4.0/22;
set_real_ip_from 141.101.64.0/18;
set_real_ip_from 108.162.192.0/18;
set_real_ip_from 190.93.240.0/20;
set_real_ip_from 188.114.96.0/20;
set_real_ip_from 197.234.240.0/22;
set_real_ip_from 198.41.128.0/17;
set_real_ip_from 162.158.0.0/15;
set_real_ip_from 104.16.0.0/13;
set_real_ip_from 104.24.0.0/14;
set_real_ip_from 172.64.0.0/13;
set_real_ip_from 131.0.72.0/22;

# Cloudflare IPv6 ranges
set_real_ip_from 2400:cb00::/32;
set_real_ip_from 2606:4700::/32;
set_real_ip_from 2803:f800::/32;
set_real_ip_from 2405:b500::/32;
set_real_ip_from 2405:8100::/32;
set_real_ip_from 2a06:98c0::/29;
set_real_ip_from 2c0f:f248::/32;

# Header that has the real visitor IP
real_ip_header CF-Connecting-IP;
```
Todo arquivo dentro de `/etc/nginx/conf.d/` é carregado para todos os sites, então você não precisa alterar o arquivo de configuração do seu site. Salve o arquivo, saia do editor de texto e como sempre teste a configuração e recarregue o nginx:
`# nginx -t && systemctl reload nginx`
Agora acesse o seu site e acompanhe o log com `# tail -f /var/log/nginx/access.log`, as suas requisições agora devem aparecer com o seu próprio endereço IP público em vez de um do Cloudflare, pressione Ctrl+C para parar de acompanhar o log.

## Atualizando a lista de IPs do Cloudflare automaticamente
Tanto o firewall quanto a configuração de IP real dependem da lista de IPs do Cloudflare, e se o Cloudflare adicionar novos endereços e nós esquecermos de atualizar o nosso servidor, os visitantes podem não conseguir acessar o site ou vão aparecer com IPs do Cloudflare nos logs. Então nós vamos fazer um script que baixa as listas, regenera as duas configurações e executá-lo todo dia com um timer do systemd.
Se você seguiu a seção opcional do firewall, primeiro precisamos mover os sets do Cloudflare para um arquivo próprio para que o script possa substituí-los, então execute `# mkdir -p /etc/nftables.d`, edite o arquivo `/etc/nftables.conf`, apague os dois sets(`cloudflare_v4` e `cloudflare_v6`) e no lugar deles, logo depois da linha `table inet filter {`, adicione esta linha:
```
        include "/etc/nftables.d/cloudflare.nft"
```
Salve o arquivo, o script vai criar o `cloudflare.nft` quando for executado, se você não seguiu a seção opcional do firewall simplesmente pule esta parte e o script vai atualizar somente o nginx.
NOTA: Tenha certeza de realmente apagar os sets antigos, a verificação `nft -c` que o script faz não vai te avisar se você esquecê-los, o nftables simplesmente junta os dois sets com o mesmo nome em um só, então os endereços antigos vão continuar no seu firewall mesmo depois que o Cloudflare removê-los da lista. Você pode verificar isso com `# nft list set inet filter cloudflare_v4` depois que o script for executado, ele deve mostrar somente os endereços da lista do Cloudflare.
Agora tenha certeza de que o curl está instalado com `# apt install curl`, e como root crie o arquivo `/usr/local/sbin/update-cloudflare-ips` com o seu editor de texto favorito e cole o seguinte conteúdo:
```
#!/bin/sh
# Downloads the Cloudflare IP lists and updates the nginx real IP
# configuration and the nftables sets (when nftables.conf includes them)
set -eu

NGINX_CONF=/etc/nginx/conf.d/cloudflare.conf
NFT_SETS=/etc/nftables.d/cloudflare.nft

TMP=$(mktemp -d)
trap 'rm -rf "$TMP"' EXIT

curl -fsS --max-time 30 -o "$TMP/v4" https://www.cloudflare.com/ips-v4
curl -fsS --max-time 30 -o "$TMP/v6" https://www.cloudflare.com/ips-v6

# Stop if a list is empty or has anything that isn't an IP range
for list in v4 v6; do
    if [ ! -s "$TMP/$list" ] || grep -qvE '^[0-9a-f.:]+/[0-9]+$' "$TMP/$list"; then
        echo "Invalid Cloudflare $list list, nothing was changed" >&2
        exit 1
    fi
done

# Generate the nginx real IP configuration
{
    echo "# Generated by update-cloudflare-ips, do not edit"
    awk '{ print "set_real_ip_from " $1 ";" }' "$TMP/v4" "$TMP/v6"
    echo "real_ip_header CF-Connecting-IP;"
} > "$TMP/nginx.conf"

if ! cmp -s "$TMP/nginx.conf" "$NGINX_CONF"; then
    cp "$NGINX_CONF" "$TMP/nginx.conf.old" 2>/dev/null || true
    cp "$TMP/nginx.conf" "$NGINX_CONF"
    if nginx -t -q; then
        systemctl reload nginx
        echo "nginx real IP configuration updated"
    else
        # Restore the previous configuration if the new one is broken
        if [ -f "$TMP/nginx.conf.old" ]; then
            cp "$TMP/nginx.conf.old" "$NGINX_CONF"
        else
            rm -f "$NGINX_CONF"
        fi
        echo "nginx test failed, previous configuration restored" >&2
        exit 1
    fi
fi

# Only touch the firewall if nftables.conf includes the Cloudflare sets
if grep -q "$NFT_SETS" /etc/nftables.conf; then
    {
        echo "# Generated by update-cloudflare-ips, do not edit"
        echo "set cloudflare_v4 {"
        echo "        type ipv4_addr"
        echo "        flags interval"
        echo "        elements = { $(paste -sd, "$TMP/v4") }"
        echo "}"
        echo "set cloudflare_v6 {"
        echo "        type ipv6_addr"
        echo "        flags interval"
        echo "        elements = { $(paste -sd, "$TMP/v6") }"
        echo "}"
    } > "$TMP/cloudflare.nft"

    if ! cmp -s "$TMP/cloudflare.nft" "$NFT_SETS"; then
        cp "$NFT_SETS" "$TMP/cloudflare.nft.old" 2>/dev/null || true
        cp "$TMP/cloudflare.nft" "$NFT_SETS"
        if nft -c -f /etc/nftables.conf; then
            systemctl reload nftables
            echo "nftables Cloudflare sets updated"
        else
            if [ -f "$TMP/cloudflare.nft.old" ]; then
                cp "$TMP/cloudflare.nft.old" "$NFT_SETS"
            else
                rm -f "$NFT_SETS"
            fi
            echo "nftables check failed, previous sets restored" >&2
            exit 1
        fi
    fi
fi
```
O script só altera um arquivo quando a lista é diferente da que já temos, e ele sempre testa a nova configuração antes de recarregar, se o teste falhar ele coloca o arquivo anterior de volta, então um download com problema nunca vai derrubar o seu site ou o seu firewall.
Salve o arquivo e torne-o executável com o seguinte comando:
`# chmod 700 /usr/local/sbin/update-cloudflare-ips`
Agora precisamos criar o serviço do systemd que executa o script, como root crie o arquivo `/etc/systemd/system/update-cloudflare-ips.service` e cole o seguinte conteúdo:
```
[Unit]
Description=Update the Cloudflare IP ranges for nginx and nftables
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/update-cloudflare-ips
```
E o timer que vai iniciar este serviço todo dia, crie o arquivo `/etc/systemd/system/update-cloudflare-ips.timer` e cole o seguinte conteúdo:
```
[Unit]
Description=Update the Cloudflare IP ranges daily

[Timer]
OnCalendar=daily
RandomizedDelaySec=1h
Persistent=true

[Install]
WantedBy=timers.target
```
O `RandomizedDelaySec` adiciona um atraso aleatório de até uma hora para que os servidores não baixem a lista todos ao mesmo tempo, e o `Persistent=true` faz o timer rodar na inicialização se o servidor estava desligado quando ele deveria ter rodado.
Agora vamos executar o script uma vez para ter certeza que ele funciona, e depois habilitar o timer com os seguintes comandos:
`# systemctl daemon-reload && systemctl start update-cloudflare-ips.service`
`# journalctl -u update-cloudflare-ips.service`
O journal vai mostrar se as configurações foram atualizadas, ou o erro se algo deu errado, se você seguiu a seção opcional do firewall verifique se os sets foram carregados com `# nft list set inet filter cloudflare_v4`, depois habilite o timer:
`# systemctl enable --now update-cloudflare-ips.timer`
Você pode verificar quando ele vai rodar da próxima vez com `# systemctl list-timers update-cloudflare-ips.timer`.
