
Introdução:
 
O projeto proposto visa o desenvolvimento de uma aplicação em .NET Core que atenda aos mais altos padrões de robustez, alta performance e eficiência operacional. A aplicação terá a responsabilidade de realizar operações complexas de leitura, manipulação e importação de dados provenientes de fontes diversas, como Oracle e arquivos .Xlsx, além de oferecer funcionalidades avançadas, como execução de procedures no SQL Server, parametrização de escalonamento, e uma interface web abrangente para monitoramento das execuções.
 
Requisitos Funcionais:
 ID	Descrição
RF01	Realizar a leitura de 20 views complexas da fonte de dados Oracle.
RF02	Realizar a inserção dos dados recuperados nas views para tabelas no SQL Server.
RF03	Possibilitar a leitura de 30 arquivos .Xlsx de uma pasta compartilhada.
RF04	Realizar a importação dos dados lidos do Excel e importar para o SQL Server.
RF05	Possuir estrutura para a execução de procedures no SQL Server conforme necessidade.
RF06	Possibilitar a execução das cargas de dados (Views/Arquivos/Procedures) de três formas: diariamente em um horário determinado, de forma isolada conforme a demanda de uma carga específica, e com base em alteração específica na base de dados que acione o processo.
RF07	Possibilitar uma interface WEB de monitoramento completo das execuções e métricas, incluindo execução manual e automática, logs completos, tempo geral de execução, tempo individual de execução e quantidade de registros processados para cada origem.
RF08	Possibilitar a exportação dos logs do processo em formatos Excel e PDF.
RF09	Possibilitar a parametrização de escalonamento da aplicação e processamento paralelo para maior performance.
 
Requisitos Não Funcionais:
 ID	Descrição
RNF01	A aplicação deverá ser desenvolvida em .NET Core 8.
RNF02	A aplicação deverá possuir uma estrutura robusta e eficiente de logs, utilizando bibliotecas como Serilog ou NLog.
RNF03	A aplicação deve ser escalável, suportando um aumento significativo no volume de dados processados.
RNF04	A aplicação deve garantir alta performance, especialmente durante processos de importação e execução de procedures.

https://drive.google.com/file/d/1Gc2t9dEBt1_dPbPw_jXiWUct9Lkyj7de/view?usp=sharing
 
Definições do projeto - BACKEND
[RF01, RF02, RF03, RF04, RF05]
1-	Desenvolver no backend duas aplicações de micro serviços, uma para os RFs de leitura e outra para os RFs de gravação, totalmente desacopladas. A aplicação de leitura importará os dados para uma base de dados intermediária independente controlada por uma terceira camada de micro serviços acessíveis apenas pelas aplicações.
	- 1 endpoint para ler view scripts Oracle [RF01]
	- 1 endpoint para ler arquivos xlsx [RF03]
	- 1 endpoint para gravar os dados na base intermediária SQL Server 
	- 1 Endpoint de leitura das tabelas da base de dados intermediária (MS-SQL), estas tabelas devem possuir basicamente as mesmas estruturas das tabelas de destino.
	- 1 Endpoint de gravação nas tabelas da base de dados intermediária (MS-SQL). Implementar serviços de mapeamentos de dados e execução de stored procedures. [RF02, RF04, RF05]
[RF06, RF07]
2- Desenvolver uma aplicação Web para acionamento e monitoramento das transações.
	- modo de execução manual
	- modo de agendamento de execução diária
	- modo de acionamento de gatilho por alteração específica
[RF08]
3- Implementar na aplicação Web o recurso de geração dos logs a partir da base de dados intermediária. Visando não afetar o desempenho das rotinas de importação de exportação, pode ser mais adequado gerar relatórios de logs a partir dos registros movimentados nas tabelas da base intermediária, ao invés de registrar cada transação durante a execução das rotinas.
[RF09] [RNF04]
4- Podemos abstrair as operações de importação e exportação de dados em classes bases virtuais que podem ser instanciadas em padrão factory de acordo com o tipo de operação (Views/Arquivos/Procedures) e em seguida pela estrutura individual das tabelas de destino (mapeamento dos campos).
- implementação de codificação específica para cada entidade: PROS- flexibilidade no tratamento específico dos campos sem afetar o comportamento das procedures padrões e sem necessidade de definir e ajustar parâmetros exclusivos para cada situação específica. CONS- necessidade de implementar novos códigos sempre que houver nova tabela ou view ou procedure adicionada ou alterada. Situação atenuada ao desacoplarmos em outras camadas de micro serviços. 
- outra opção seria desenvolver um módulo de configuração de mapeamento de tabelas, campos e regras de importação em uma estrutura estática de base de dados local.
 
Modelo micro serviços desacoplados
 
Projeto_01.png

Bibliotecas
- Autenticação : API proprietária 
- Importação de planilhas XLSX : 
- EPPlus (OfficeOpenXml): não precisa do MSOffice, open source, mas é um pouco mais lenta.
- ClosedXML.Excel: maior performance, porém mais simples, não suporta funções avançadas do Excel
- Interop.Excel: suporta funções avançadas, porém requer o MSOffice instalado e é mais lenta.
- Importação de dados do Oracle :
- ODP.NET (Oracle Data Provider): biblioteca oficial da Oracle para o .NET, permite executar comandos SQL, substituiu o Oracle.DataAccess.Client, alto desempenho.
- EntityFramework: ORM para .NET que pode ser usado também para Oracle. Porém por usar LINQ pode reduzir drasticamente a performance.
- Dapper: Também uma ORM para .NET, pode ser usada com o Oracle e é ideal para operações mais complexas, e para execução de stored procedures.
- Mapeamento de tabelas SQL : 
	- ADO DB
	- Extensions.Configuration.JSON
- Execução em Segundo Plano
- Quartz.NET: Ferramenta para execução de tarefas em segundo plano sem necessidade de recorrer a Windows Services, além de ser altamente customizável e robusta.
Bancos NO-SQL:
- MongoDB: Seria usado no serviço de repositório para armazenar e organizar os registros importados, por ser altamente escalável poderia ainda ser usado para armazenar logs e histórico de dados.
- ElasticSearch - Kibana
