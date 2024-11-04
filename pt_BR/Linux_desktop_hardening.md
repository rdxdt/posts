# 0x00 - Aviso de isenção de responsabilidade
Eu não sou um expert certificado em segurança, estas dicas são somente coisas que eu aprendi com meus aproximadamente 20 anos pesquisando, e eu escrevo este artigo em boa fé na esperança de que será útil para alguem.

# 0x01 - Segurança x Privacidade

Enquanto estes são dois conceitos diferentes, eles se sobrepoem, alguns levam para o extremo e fazem deles mesmos um fantasma, um pouco de privacidade pode ajudar bastante em sua segurança, mas privacidade não é o foco principal deste post.
Com isso em mente nós precisamos definir nosso objetivo, e eu não me importo de compartilhar o meu e eu penso que o seu poderá ser parecido; meu objetivo principal é preservar o meu bem estar, e isso inclui fazer com que uma potencial ameaça não consiga ter um impacto negativo significante na minha qualidade de vida ou saúde; e isso é bem vago e se sobrepõe com a minha segurana física.

# 0x02 - Segurança por obscuridade

Este conceito é o maior abismo que costumamos cair, mas isto é exagerado com frequencia, armazenar suas senhas em uma maneira segura não constitui segurança por obscuridade, mas trocando a porta que o seu servidor SSH usa, pode ser considerado segurança por obscuridade, pois por si só não faz as coisas mais seguras.

# 0x03 - Conheça seu adversário

De quem você está tentando se proteger? Esta é a questão que você precisa pensar sobre, porque ela é imperativa para se saber quais medidas serão suficientes, é mais facil lidar com um script-kiddie assistindo vídeos no youtube sobre como usar as ferramentas do kali linux, do que lidar com agências de inteligências e companias israelense de desenvolvimento de spyware.

# 0x04 - Preparando uma boa fundação para a segurança​

Já ouviu que “a sua segurança é tão boa quanto seu elo mais fraco?”, se a sua base é o Windows 10 ou 11, então você está tendo um mal começo, a magia do software livre e código aberto, é a capacidade de audita-lo, mas eu vou te dizer um segredo, não conte para ninguém certo? Eu não leio todas as linhas de código que são compiladas e executada no meu sistema.
Então é muito importante que você tenha um sistema operacional que rode software de uma fonte que você confia, pessoalmente, eu confio no time do openSUSE, também no time do Debian, então openSUSE e Debain são frequentemente minhas escolhas para sistemas operacionais, mas entenda que isto é só um exemplo, outras distribuições 'convencionais' tem um bom histórico de segurança, mas incidentes de segurança podem acontecer como foi o caso de [quando o Linux Mint foi atacado](https://arstechnica.com/information-technology/2016/02/linux-mint-hit-by-malware-infection-on-its-website-and-forum-after-hack-attack/), [ou a vez que os servidores do Debian foram hackeados](https://www.linuxinsider.com/story/hackers-compromise-debian-linux-project-servers-58264.html) , ou até mesmo [este incidente com a RedHat](https://www.zdnet.com/article/red-hats-ceph-and-inktank-code-repositories-were-cracked/)

# 0x05 - Força da senha e Autenticação de 2 fatores

Por favor não use senhas óbvias como nomes, aniverśarios, ou senhas padrões como a senha padrão da interface de administração web.
Também armazenando senhas em uma maneira segura é recomendável, a minha recomendação é ter uma senha forte que você memorizou como uma senha mestra para um gerenciador de senhas como o KeePassXC ou BitWarden.
Esteja ciente que serviços comerciais que prometem armazenar suas senhas são perigosos, se eles forem atacados, isto poderá ser catastrófico para você.
Se você precisa de usar suas senhas em multiplos dispositivos, você pode armazenar seu banco de dados do KeePassXC em um pendrive(se você quer ter uma segurança extra você pode utilizar criptografia de disco naquele pendrive).
Também como uma medida de segurança tenha um backup atualizado daquele banco de dados.
Outro aspecto importante é habilitar a autenticação de dois fatores sempre que possível, mesmo com chaves de hardware como uma Yubi Key, contudo o aplicativo Google Authenticator pode ser útil.

# 0x06 - Hashes e Assinaturas, eles são um dos seus melhores amigos​

Vamos assumir que você não confia mais no seu sistema operacional e pensa que ele pode estar comprometido porque você não verificou seus hashes e assinaturas e você quer começar do zero, então você faz o download de uma imagem iso, grava no seu drive usb e você está pronto para inicializar o instalador, mas calma lá, nós precisamos de verificar se o que você baixou é o que você pretendia baixar e não está comprometido, geralmente as distribuições dão instruções específicas em como verificar o seu download, por exemplo no [guia de instalação do Linux Mint](https://linuxmint-installation-guide.readthedocs.io/en/latest/verify.html) tem uma seção bem detalhada em como verificar, o Debian também provê um [guia de como verificar a sua ISO](https://www.debian.org/CD/verify), o mesmo pode ser dito para o [Arch Linux em sua wiki](https://wiki.archlinux.org/title/Installation_guide); Então eu creio que você já entendeu.
Agora que você verificou, vamos instalar o sistema e é ai que o nosso próximo ponto se torna importante.

# 0x07 - Secure Boot​

Algumas distribuições fazem a implantação em plataformas com Secure Boot bem fáceis. Na minha experiência o Debian e o openSUSE são exemplos que fazem a instalação em sistemas com secure boot habilitado bem simples, sem necessidade de intervenção do usuário, o Arch Linux entretanto [requer intervenção do usuário](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot).
Enquanto o Secure Boot tem suas falhas, os benefícios são maiores que os prejuízos, e algumas das falhas podem ser mitigadas ao substituir as Plaform Keys, mas isso tem o risco de inutilizar seu sistema, então não me aprofundarei neste assunto.
Também lembre-se de definir uma senha para acessar o utilitário de configuração da firmware UEFI, pois senão alguém poderá desabilitar o Secure Boot e você não perceberá, tornando você vulnerável.

# 0x08 - Criptografia completa de disco.

Isso pode parecer desnecessário, mas confie em mim, isto é muito importante, é aonde a cyber segurança encontra a segurança física, grande parte das distribuições fáceis de instalar, ofereção a opção de habilitar a criptografia completa de disco no momento da instalação, até mesmo o utilitário de instalaçãoi do Arch Linux chamado archinstall, o openSUSE por sua vez vai além e permite criptografar a sua partição de boot, porém a opção de configurar o LUKS manualmente é universal.
Se você não quer digitar uma senha longa muitas vezes, você pode utilizar um módulo TPM com algumas configurações adicionais.
Fique ciente que a maior proteção dos dados que a criptografia completa de disco fornece é quando o sistema está desligado, portanto se você vai deixar o seu dispositivo sozinho por um tempo, é legal que você o desligue se você está em um local que a sua segurança física não garanta que seu dispositivo não estará acessível para mais ninguém.

# 0x09 - Firewalling​

Nome legal hein? Enquanto por padrão nenhuma distribuição convencional de linux vem com um firewall de aplicativos, ter um firewall de rede habilitado pode ser bem legal, alguns frontends como ufw e gufw tornam simples a configuração.
Certifique-se de que a política de conexões de entrada é definida para DROP em tudo que não é permitido por uma regra.
Firewalls são ferramentas bem versáteis que permitem a realização de várias coisas interessantes, você pode até ir em um modo extra paranóico e fazer que a sua política de conexões de saída é DROP e fazer regras apenas para portas críticas como a TCP 80(HTTP), TCP 443(HTTPS), UDP 53(DNS), mas fique ciente das consequencias das suas regras, outros aplicativos podem depender de outras portas.
A regra de ouro é saber o que você precisa.

# 0x0A - Protegendo o seu navegador web

Isto é bem importante, websites podem ser vistos não apenas como um mecanismo de entrega de conteúdo, mas também como um mecanismo de entrega de código, é de suma importância que você mantenha o seu navegador atualizado e muito bem configurado.
Pessoalmente eu uso o Mozilla Firefox, e tenho meu perfil e configurações com um bom backup, mas se você está começando do zero, o [FFProfile](https://ffprofile.com/)  pode ser bem útil para lhe ajudar com a criação de perfil de configuração;
Algumas extensões que eu recomendo são: uBlock Origin, Decentraleyes, ClearURL, CanvasBlocker(com sua configuração stealthy) e a Privacy Badger.
Certifique-se de configurar seu navegador para usar somente HTTPS(ou utilize a extensão HTTPS Everywhere, você receberá alguns avisos em websites que não implementaram SSL, mas é um ótimo modo de te avisar quando você está entrando em um site potencialmente inseguro, portanto evite inserir dados pessoais).
Você pode ou não optar por usar um trocador de user agent, mas é importante reconhecer suas limitações, e estar ceiente que até mesmo com um trocador de user agent, websites podem identificar o verdadeiro núcleo do seu navegador.
Um bom local de teste é o [website DeviceInfo](deviceinfo.me).
Se você está interessado em aprimorar e ter um controle mais granular, você pode utilizar o addon NoScript.
Outra coisa para se considerar é que isso é um ponto que a privacidade conflitua com a segurança, essas extensões vão auxiliar com a segurança, mas como as configurações e extensões desviam do padrão da maior parte dos usuários da internet, isso fará seu navegador se sobressair.
É importante mencionar que navegadores como Chromium, Ungoogled Chromium, Brave e Vivaldi existem e podem ser configurados para uma melhor segurança.
Breve atualização: A maioria dos navegadores baseados no Chromium estão sofrendo com a depreciação do manifest v2 e o manifest v3 quebra os bloqueadores de propaganda mais poderosos, tente utilizar o Brave se você precisa de um navegador baseado no Chromium ou fique com Firefox.

# 0x0B - X11/Xorg vs Wayland​

Este item causará algumas discussões, Wayland sendo a 'nova criança na escola' poderá causar alguns problemas, mas seus protocolos tem bem mais considerações de segurança.
Sendo que o X11 está a bem mais tempo em uso, a maioria das aplicações suportam-o, mas em contrapartida no X é mais fácil para aplicativos maliciosos monitorarem sua tela, seu teclado e mouse.
Com GPU da NVIDIA, o Wayland pode ser meio problemático(mas está melhorando), mas é basicamente isso, no Wayland você pode utilizar Xwayland para rodar aplicações feitas para o X.
Então é melhor estar em uma sessão Wayland.

# 0x0C - Sandboxing​

Seguindo a regra de que tudo deverá ter acesso somente ao necessário, rodar um aplicativo em uma sandbox pode auxiliar a conseguir este objetivo, existem duas alternativas famosas para sandboxing, que são bubblewrap(utilizado no Flatpak) e Firejial, Firejail é bem interessante, vem com perfis prontos, mas você provavelmente precisará de personalizar e realizar algumas configurações manuais, então pode ser que seja mais atraente utilizar flatpaks e personalizar os privilégios com o flatseal, você pode até ter alguns flatpaks e utilizar o firejail para aplicativos que não foram instalados com flatpak.
Lembra que eu disse na seção sobre firewall que as distribuições linux não vem com um firewall de aplicativos? Com sandboxing você pode mitigar este problem, você pode negar acesso a rede para certos aplicativos.
Apenas fique atento as restrições das sandboxes, elas podem ficar no caminho e parecer como um bug.

# 0x0D - Mandatory Access Control(SELinux/AppArmor)​

E se um aplicativo confiável é explorado e um  atacante consegue injetar código arbitrário dentro dele?
M.A.C pode ajudar a mitigar os danos, limitando o que o aplicativo tem acesso, então verifique se o seu sistema tem isto configurado e está sendo exigido.
Como uma má configuração de M.A.C pode causar todos os tipos de problemas, eu recomendo uma boa leitura e entendimento antes de exigir estas regras.
Ambas as [Arch wiki](https://wiki.archlinux.org/title/AppArmor) e [Debian wiki](https://wiki.debian.org/AppArmor) tem bons artigos sobre AppArmor, e a [RedHat entra em detalhes sobre M.A.C e SELinux]( https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/using_selinux/getting-started-with-selinux_using-selinux), [o openSUSE também tem um bom artigo sobre configuração do SELinux](https://en.opensuse.org/SDB:SELinux), entretanto fique atento que o [SELinux está disponível para Arch Linux](https://wiki.archlinux.org/title/SELinux) mas não é oficialmente suportado.

# 0x0E - Aprimorando sysctl.conf ​

Algumas configurações no kernel podem ser aprimoradas para um pouco mais de proteção, mas elas podem interferir com algumas funcionalidades.
Pessoalmente eu utilizo algumas configurações de prevenção de 'man in the middle' e desabilito coisas que eu não preciso.
Um pouco mais sobre pode ser lido nos links a seguir:

- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/kernel_administration_guide/working_with_sysctl_and_kernel_tunables
- https://www.cyberciti.biz/faq/linux-kernel-etcsysctl-conf-security-hardening/ 
- https://geektnt.com/sysctl-conf-hardening.html
- https://linoxide.com/how-tos/linux-server-protection/
- https://github.com/klaver/sysctl/blob/master/sysctl.conf

# 0x0F - Senso comum e fontes de software

Na minha experiência, esta é a maior fonte dos problemas, não apenas relacionados a segurança .
Por favor evite o máximo possível utilizar repositórios de terceiros, e quando você tiver que usar, como por exemplo no openSUSE que não vem com os codecs multimídia proprietários e você precisa utilizaro repositório do packman para estes, mas essa é uma exceção pois este é um projeto confiável que realmente melhora a usabilidade do openSUSE, outro exemplo de um repositório de terceiro confiável é o repositório do Wine, entretanto possibilitar a execução de executaveis do Windows, abrem a porta para a execução de malware para Windows, e por padrão o Wine expõe boa parte do seu sistema e permitirá a exfiltração de dados. 
A não ser que você saiba o que está fazendo, evite rodar binários aleatórios ou AppImages adiquiridos de fontes não confiáveis, e isso se aplica a scripts, sejam eles shell scripts, scripts de perl, scripts de python.
Pacotes do npm do node.js e Pypi do python são frequentemente visados e já tiveram algumas vítimas, então tenha cuidado!
Também evite utilizar pipes para o bash.

# 0x10 - Patch Patch Patch​

Mantenha seu sistema atualizado, isso não ajuda muit com exploits 0day, mas alguns destes são divulgados responsavelmente para os desenvolvedores antes de serem publicados, portanto uma atualização pode conter a correção para um exploit divulgado para os desenvolvedores que serão publicados amanhã.
Isso é um pouco mais complicado em serviores, mas para seu computador pessoal, isso não deverá trazer muitos problemas, a não ser que você esteja usando Arch Linux, neste caso antes de atualizar, de uma boa lida nos fóruns e as notícias antes de atualizar.

# 0x11 - Backups (Criptografados)​

Isto é muito importante, enquanto alguns sistemas de arquivos como btrfs permitem você ter snapshots e certos níveis de RAID podem tolerar alguma falha de hardware, estes não são substitutos para backups reais, se você tem um NAS, é bem conveniente ter um backup nele, mas você também deve considerar ter um backup totalmente offline, em casa é interessante ter um HD externo, mas ter equipamentos para backups só é tão bom quanto a sua agenda de backups.
Algumas pessoas podem estar satisfeitas com backups diários, outras com backups semanais e outras com backups mensais.
Para as suas coisas críticas, você pode considerar comprimir elas em um tarball e utilizar gpg para criptografar e manter uma cópia em outro local, como seu VPS ou serviços de armazenamento na núvem tradicionais.
Só não faça upload das suas chaves privadas.

# 0x12 - Saiba das limitações de Proxies e "VPN" e atente-se ao TOR​

Enquanto existem vários youtubers promovendo VPN e companias de VPN com propagandas que prometem um túnel secreto que irá te tornar anônimo e super seguro, na realidade isto não passa de um exagero de marketing.
Claro que um túnel fara com que seu provedor de internet saber apenas que você está conectado ao túnel, e o destino saberá que a conexão está vindo do túnel, o provedor de VPN saberá seu tráfego, e eles podem coletar logs(incluindo aqueles que dizem não coletar logs por padrão, mas um pedido de um juiz é suficiente para fazerem eles coletarem logs de alguém e entregar para a justiça), e seu pagamento.
VPN “gratuitas” são perigosas, se você for sortudo você só tera uma largura de banda limitada e um limite de dados que o provedor de VPN utilizará para te atrair para a versão paga, mas se você não for sortudo, você será espionado, ou pior, seu computador sendo utilizado como proxy residencial.
Entretanto o TOR é bem interessante, mas você precisa de ser cuidadoso, e eu não entrarei em muitos detalhes, mas é importante saber que todos os nós de entrada e saída são públicamente conhecidos, e ataques de correlação são bem populares.