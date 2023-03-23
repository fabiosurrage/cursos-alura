## VPC

A VPC, ou Virtual Private Cloud (em português, Nuvem Virtual Privada), é uma parte muito importante da computação na AWS e em outros serviços de cloud: ajuda a separar aplicações com uma camada a mais de isolamento e protege os dados de aplicações, além de permitir uma proteção extra para a aplicação, ao utilizar redes privadas.

A VPC é um recurso complexo com várias partes que devem se comunicar perfeitamente, que são:

Redes públicas
Redes privadas
Internet gateway
NAT gateway
Route Tables
VPC
Nas redes públicas encontram-se todos os recursos criados que ganham um endereço de IP público, ou seja, que se conectam diretamente à internet. Porém, esse IP pode mudar quando ligamos e desligamos as máquinas. Se queremos um IP público fixo, podemos usar o Elastic IP, que é pago. Esse tipo de rede é recomendada para os load balancers, e também se a nossa aplicação recebe as requisições diretamente dos clientes; ou em casos de alguns recursos específicos, como o NAT gateway.

Nas redes privadas, todas as máquinas têm um IP privado que só pode ser usado dentro da própria VPC, dando assim uma camada a mais de segurança para as máquinas, já que não estão conectadas diretamente na internet. Para acessarmos essas máquinas via SSH, é necessário usar uma máquina pública como ponte de acesso. Nessa rede, colocamos as nossas aplicações, podendo ser instâncias do EC2, instâncias docker, como o ECS ou o beanstalk, e as instâncias do kubernetes.

O internet gateway permite que os recursos na rede pública possam acessar à internet como se fosse o modem que a operadora instala nas casas, fazendo a ligação das redes com a internet.

O NAT gateway é um recurso que deve ficar na rede pública para receber todas as requisições que vêm da rede privada e enviá-las para o internet gateway, como se fosse um roteador de uma casa, interligando a sua rede ao modem (muitos aparelhos hoje já tem as duas função de modem e roteador; então se você tiver apenas um aparelho em casa saiba que ele faz as duas funções).

Como decidir para onde enviar as requisições? Esse é o trabalho da Route Table, que faz a interligação entre os recursos e permite que tudo se comunique. Então, a rede privada vai ter como alvo o NAT gateway, e a rede pública o internet gateway.

A VPC é o recurso que vai conter todos os outros e permitir com que eles trabalhem em sincronia, cada um realizando uma tarefa.

São vários recursos diferentes. Muitas vezes, temos que criar múltiplos de cada recurso, como as redes públicas e privadas, uma em cada zona de disponibilidade, e garantir assim uma maior resiliência da aplicação.

## Modulos Terraform

Os módulos são partes muito importantes dentro do Terraform: definimos ambientes, reutilizamos códigos e criamos blocos de infraestrutura; porém eles também têm outras utilidades, como distribuir o seu código com a comunidade, ou usar trechos de código desenvolvidos pela comunidade.

Os módulos desenvolvidos pelos provedores são devidamente testados e têm uma chance muito baixa de erros, tornando nossa infraestrutura mais segura e com menor tempo de desenvolvimento. Geralmente esses módulos criam blocos com vários recursos que são dependentes entre si, como é o caso da VPC.

Podemos buscar pelos módulos na documentação do terraform, dando sempre prioridade para aqueles desenvolvidos pelos provedores, como a AWS e o Google Cloud, e para módulos com grandes quantidades de downloads.

Lembrando que os módulos executam código e por motivos de segurança não devemos executar códigos de fontes desconhecidas, então sempre que for usar um módulo com um número baixo de downloads (algo por volta de 10 mil) é bom dar uma olhada no código fonte dele para se certificar que nada está errado.

Também é possível usar outras fontes para os módulos além dos descritos na documentação. Porém, deve-se tomar um cuidado ainda maior com esses, pois não é certo que passaram por checagens, tanto de desempenho, como de compatibilidade e segurança.

Para usar os módulos que estão fora do registro oficial, podemos dar uma olhada na documentação sobre módulos, onde encontramos como usar repositórios do Github, Bitbucket ou de repositórios Git e Mercurial, endereços HTTP, buckets do S3 entre outros.

## Filtrando Entradas com Security Groups

Precisamos definir de onde podem vir as requisições e conexões para as máquinas quando criamos os grupos de segurança. Em muitos casos, como nas redes privadas, não queremos receber as requisições e conexões diretamente da internet, por questões de segurança. Então, o grupo de segurança deve fazer o papel de um firewall, filtrando o que pode e o que não pode passar.

Temos dois jeitos de especificar quais as fontes essas requisições podem alcançar nas nossas máquinas: pelo CIDR, para endereços de IPs, ou vindo de outros grupos de segurança.

Sempre que queremos receber as requisições diretamente da internet, usamos o CIDR, ele representa uma lista de endereços de IP, podendo ter um único item até $255^{4}$ membros (pouco mais de 4.2 bilhões de membros). Diante disso, imagine escrever uma lista com 4.2 bilhões de endereços?! Ao invés de escrevermos essa lista, usamos uma notação feita para o CIDR, em que representamos um bloco através do IP de início, seguido de uma barra e quantos bits podem ser substituídos.

Ao escrevermos 10.0.0.0/24, temos uma lista na qual os últimos 8 bits podem ser trocados, indo de 10.0.0.0 até 10.0.0.255. Se escrevermos 192.168.0.0/23, temos uma lista na qual os últimos 10 bits podem ser trocados, indo de 192.168.0.0 até 192.168.0.255 e de 192.168.1.0 até 192.168.1.255. Os conjuntos mais comuns são 0.0.0.0/0 que servem para todos os endereços existentes, 10.0.0.0/16 e mais alguns; você pode encontar uma lista completa para os endereços IPv4 e IPv6 aqui.

A outra opção é definir um grupo de segurança como a entrada, assim as máquinas só podem receber requisições de outros recursos criados por você, como um load balancer, o que oferece uma segurança maior.

Outro exemplo é caso tenhamos instâncias que realizam cálculos ou que contenham um banco de dados e que não são instâncias de uma API ou um servidor web. Por essas instâncias terem informações sensíveis, não podemos deixá-las abertas para a internet. Entretanto, caso queiramos acessar via ssh, podemos usar uma máquina na rede pública, e aumentar assim a segurança; e mesmo que a máquina seja comprometida, o banco de dados está protegido, pois não podemos fazer requisições diretas para ele.

