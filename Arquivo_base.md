www.unirv.edu.br

TÉCNICO EM INFORMÁTICA
BANCO DE DADOS II
PROF. JÚLIO CÉSAR

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PLANO DA AULA PRÁTICA SISTEMAS
INTEGRIDADE DE DADOS
1
CHECK, PK, FK e TRIGGER
SEGURANÇA EM BANCO DE DADOS
2
CREATE USER, GRANT, REVOKE, ROLE
PRIVACIDADE E LGPD
3
AES, hash, auditoria e direitos
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
COMO VAMOS TRABALHAR SISTEMAS
Aula 100% prática, do começo ao fim:
• Digite, execute e observe cada resultado
• Vamos provocar erros de propósito
• Scripts prontos no arquivo Word da aula
• Em duplas: um digita, o outro confere
Ambiente: MySQL 8.0.16+ (Workbench ou terminal)
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PREPARANDO O AMBIENTE SISTEMAS
DROP DATABASE IF EXISTS escola;
CREATE DATABASE escola
CHARACTER SET utf8mb4;
USE escola;
SELECT VERSION();
Use MySQL 8.0.16+: só assim o CHECK é aplicado.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
BLOCO 1 — INTEGRIDADE SISTEMAS
DOMÍNIO ENTIDADE
A B
CHECK, NOT NULL e PRIMARY KEY e
DEFAULT UNIQUE
REFERENCIAL DEFINIDA PELO
C D
FOREIGN KEY e USUÁRIO
CASCADE CHECK e TRIGGER
~85 minutos, com um desafio no final.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SEM INTEGRIDADE — O PROBLEMA SISTEMAS
CREATE TABLE sem_regras (
id INT, idade INT, email VARCHAR(100)
);
INSERT INTO sem_regras VALUES
(1, -5, NULL), (1, 200, 'x');
SELECT * FROM sem_regras;
O banco aceitou tudo: id repetido, idade absurda e e-mail nulo.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DOMÍNIO — CRIANDO A TABELA SISTEMAS
CREATE TABLE aluno (
idade INT CHECK (idade BETWEEN 14 AND 99),
email VARCHAR(100) NOT NULL,
status VARCHAR(10) DEFAULT 'ATIVO',
matricula CHAR(10)
CHECK (matricula LIKE 'BD2%')
);
Quatro regras de domínio: CHECK, NOT NULL, DEFAULT e
LIKE.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DOMÍNIO — TESTANDO (1/2) SISTEMAS
-- OK: tudo válido
INSERT INTO aluno (idade, email, matricula)
VALUES (17, 'ana@escola.com', 'BD2-001');
-- ERRO: idade fora do intervalo
INSERT INTO aluno (idade, email, matricula)
VALUES (200, 'x@x.com', 'BD2-002');
O CHECK barrou a idade 200 (erro 3819).
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DOMÍNIO — TESTANDO (2/2) SISTEMAS
-- ERRO: e-mail obrigatório
INSERT INTO aluno (idade, email, matricula)
VALUES (17, NULL, 'BD2-003');
-- ERRO: matrícula fora do padrão
INSERT INTO aluno (idade, email, matricula)
VALUES (17, 'y@y.com', 'XYZ-004');
SELECT * FROM aluno;
Só a linha válida foi gravada, com status ATIVO.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DOMÍNIO — REGRA COM ALTER SISTEMAS
ALTER TABLE aluno ADD CONSTRAINT chk_email
CHECK (email LIKE '%@%');
-- ERRO: e-mail sem @
INSERT INTO aluno (idade, email, matricula)
VALUES (17, 'semarroba', 'BD2-005');
-- Próximo passo: recomeçar a tabela
DROP TABLE aluno;
ALTER TABLE adiciona regras a uma tabela já criada.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
MÃO NA MASSA SISTEMAS
UNIDADE 1 — RESTRIÇÃO DE DOMÍNIO
Exercício orientado
Crie a tabela produto: nome obrigatório, preço maior
1
que 0 e estoque com valor padrão 0. Insira um
produto válido e provoque 2 erros.
Resolução individual, com correção coletiva ao final.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CORREÇÃO — TABELA PRODUTO SISTEMAS
CREATE TABLE produto (
id_produto INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(80) NOT NULL,
preco DECIMAL(10,2) CHECK (preco > 0),
estoque INT DEFAULT 0
);
-- ERRO esperado: preço zero
INSERT INTO produto (nome, preco)
VALUES ('Lápis', 0);
Também vale testar nome nulo e um produto válido.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
ENTIDADE — CRIANDO A TABELA SISTEMAS
CREATE TABLE aluno (
id_aluno INT PRIMARY KEY AUTO_INCREMENT,
cpf CHAR(11) UNIQUE NOT NULL,
nome VARCHAR(80) NOT NULL,
idade INT CHECK (idade BETWEEN 14 AND 99),
email VARCHAR(100) NOT NULL,
status VARCHAR(10) DEFAULT 'ATIVO'
);
PRIMARY KEY identifica a linha; UNIQUE protege o CPF.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
ENTIDADE — TESTANDO O UNIQUE SISTEMAS
INSERT INTO aluno
(cpf, nome, idade, email) VALUES
('11111111111', 'Ana', 16, 'a@e.com'),
('22222222222', 'Bia', 17, 'b@e.com');
-- ERRO: CPF repetido (UNIQUE)
INSERT INTO aluno
(cpf, nome, idade, email) VALUES
('11111111111', 'Caio', 18, 'c@e.com');
Erro 1062: Duplicate entry para o CPF já cadastrado.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
ENTIDADE — TESTANDO A CHAVE SISTEMAS
-- ERRO: id_aluno 1 já existe
INSERT INTO aluno
(id_aluno, cpf, nome, idade, email)
VALUES
(1, '33333333333', 'Caio', 18, 'c@e.com');
-- AUTO_INCREMENT: veja os ids gerados
SELECT id_aluno, nome FROM aluno;
A chave primária nunca se repete e nunca é nula.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CHAVE COMPOSTA — CRIANDO SISTEMAS
CREATE TABLE turma (
id INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(50) NOT NULL
);
CREATE TABLE matricula (
id_aluno INT, id_turma INT,
PRIMARY KEY (id_aluno, id_turma)
);
O par aluno + turma não pode se repetir.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CHAVE COMPOSTA — TESTANDO SISTEMAS
INSERT INTO turma (nome)
VALUES ('Info 2A'), ('Info 2B');
INSERT INTO matricula VALUES (1,1), (2,1);
-- ERRO: o par (1,1) já existe
INSERT INTO matricula VALUES (1,1);
-- OK: mesmo aluno, outra turma
INSERT INTO matricula VALUES (1,2);
Matrículas atuais: (1,1), (2,1) e (1,2).
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
REGRA DE NEGÓCIO — CHECK SISTEMAS
CREATE TABLE funcionario (
id_func INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(80) NOT NULL,
salario DECIMAL(10,2)
CHECK (salario >= 1518.00)
);
Valor do material da aula: ajuste ao salário mínimo vigente.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
REGRA DE NEGÓCIO — TESTANDO SISTEMAS
INSERT INTO funcionario (nome, salario)
VALUES ('Carla', 2500.00);
-- ERRO: abaixo do salário mínimo
INSERT INTO funcionario (nome, salario)
VALUES ('Davi', 1000.00);
Regras de negócio simples cabem em um CHECK.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
REGRA DE NEGÓCIO — EMPRÉSTIMO SISTEMAS
CREATE TABLE emprestimo (
id INT PRIMARY KEY AUTO_INCREMENT,
livro VARCHAR(80) NOT NULL,
data_retirada DATE NOT NULL,
data_devolucao DATE
);
Vamos proteger a regra de datas com um TRIGGER.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
REGRA DE NEGÓCIO — TRIGGER SISTEMAS
Uma trigger (ou gatilho) em SQL é um bloco de código
procedural associado a uma tabela ou visão que é
executado automaticamente
pelo banco de dados sempre que um evento
específico ocorre.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
| REGRA DE NEGÓCIO — |     | TRIGGER |     |     |     | SISTEMAS |
| ------------------ | --- | ------- | --- | --- | --- | -------- |
Uma trigger é estruturada em 4 pontos essenciais:
Tabela alvo: Onde a ação acontece.
| Evento:  | A operação executada ( |        |        | ,                       | ou     | ).  |
| -------- | ---------------------- | ------ | ------ | ----------------------- | ------ | --- |
|          |                        |        | INSERT | UPDATE                  | DELETE |     |
| Momento: | Quando                 | roda — |        | (antes, para validar),  |        |     |
BEFORE
| AFTER (depois, para registrar/auditar) ou |     |     |     |     |     |     |
| ----------------------------------------- | --- | --- | --- | --- | --- | --- |
(no lugar da ação original).
INSTEAD OF
| Frequência: | Quantas vezes |     | roda — | por cada linha modificada |     |     |
| ----------- | ------------- | --- | ------ | ------------------------- | --- | --- |
( ) ou apenas uma vez pelo comando inteiro
FOR EACH ROW
| ( FOR EACH STATEMENT |     | ).  |     |     |     |     |
| -------------------- | --- | --- | --- | --- | --- | --- |
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
TRIGGER — PARTE 1/2 SISTEMAS
DELIMITER $$
CREATE TRIGGER valida_devolucao
BEFORE INSERT ON emprestimo
FOR EACH ROW
BEGIN
O DELIMITER $$ permite usar ; dentro do trigger.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
TRIGGER — PARTE 2/2 SISTEMAS
IF NEW.data_devolucao <
NEW.data_retirada THEN
SIGNAL SQLSTATE '45000'
SET MESSAGE_TEXT = 'Data inválida';
END IF;
END$$
DELIMITER ;
O MySQL não tem ASSERTION: use CHECK e TRIGGER.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
TRIGGER — TESTANDO SISTEMAS
-- OK: devolução depois da retirada
INSERT INTO emprestimo VALUES
(NULL, 'Dom Casmurro',
'2026-09-01', '2026-09-15');
-- ERRO: devolução antes da retirada
INSERT INTO emprestimo VALUES
(NULL, 'Quincas Borba',
'2026-09-10', '2026-09-01');
Mensagem esperada: Data inválida (erro 1644).
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
RESUMO DO BLOCO 1 SISTEMAS
| RESTRIÇÃO   | COMANDO         | ERRO PROVOCADO    |
| ----------- | --------------- | ----------------- |
| DOMÍNIO     | CHECK, NOT NULL | idade 200         |
| ENTIDADE    | PK, UNIQUE      | CPF repetido      |
| REFERENCIAL | FK, CASCADE     | turma inexistente |
| USUÁRIO     | CHECK, TRIGGER  | devolução antes   |
Em todos os casos, o SGBD rejeita antes de gravar.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DESAFIO DO BLOCO 1 SISTEMAS
UNIDADE 1 — INTEGRIDADE DE DADOS
Exercício orientado
Reescreva a tabela pedido com 4 restrições: chave
1
primária, chave estrangeira, CHECK no valor e
DEFAULT no status.
Resolução individual, com correção coletiva ao final.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PEDIDO — PONTO DE PARTIDA SISTEMAS
CREATE TABLE cliente (
id_cliente INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(80) NOT NULL
);
CREATE TABLE pedido_sem_regras (
id INT, id_cliente INT, data_pedido DATE,
valor DECIMAL(10,2), status VARCHAR(15)
);
Reescreva pedido_sem_regras com 4 restrições.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CORREÇÃO — PEDIDO COM REGRAS SISTEMAS
CREATE TABLE pedido (
id_pedido INT PRIMARY KEY AUTO_INCREMENT,
id_cliente INT NOT NULL,
data_pedido DATE NOT NULL,
valor DECIMAL(10,2) CHECK (valor > 0),
status VARCHAR(15) DEFAULT 'ABERTO',
FOREIGN KEY (id_cliente)
REFERENCES cliente(id_cliente)
);
PK, FK, CHECK e DEFAULT em uma só tabela.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SISTEMAS
INTERVALO
15 minutos
Salve os scripts do Bloco 1
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
BLOCO 2 — SEGURANÇA SISTEMAS
~75 minutos, em quatro etapas:
SQL INJECTION
1
~10 min — hash e prepared statement
AUTENTICAÇÃO
2
~12 min — CREATE USER e política de senha
AUTORIZAÇÃO E DAC
3
~23 min — GRANT, REVOKE, GRANT OPTION
MAC E RBAC
4
~30 min — view por nível, ROLE e desafio
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SQL INJECTION — PREPARANDO SISTEMAS
CREATE TABLE usuario (
id INT PRIMARY KEY AUTO_INCREMENT,
login VARCHAR(30) UNIQUE NOT NULL,
senha_hash CHAR(64) NOT NULL
);
INSERT INTO usuario (login, senha_hash)
VALUES ('ana', SHA2('ana#123', 256)),
('admin', SHA2('Adm!n#2026', 256));
Guardamos o hash da senha, nunca a senha em texto.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SQL INJECTION — O ATAQUE SISTEMAS
-- login digitado: ana
SELECT * FROM usuario
WHERE login = 'ana';
-- login digitado: ' OR '1'='1
SELECT * FROM usuario
WHERE login = '' OR '1'='1';
A consulta montada pelo sistema devolve TODOS os usuários.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SQL INJECTION — A DEFESA SISTEMAS
PREPARE stmt FROM
'SELECT id, login FROM usuario
WHERE login = ?';
SET @login = "' OR '1'='1";
EXECUTE stmt USING @login;
SET @login = 'ana';
EXECUTE stmt USING @login;
DEALLOCATE PREPARE stmt;
Com parâmetro, o ataque vira só um login inexistente.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTENTICAÇÃO — CRIANDO USUÁRIO SISTEMAS
CREATE USER 'joao'@'localhost'
IDENTIFIED BY 'SenhaForte#2025';
ALTER USER 'joao'@'localhost'
IDENTIFIED BY 'NovaSenhaForte#2025';
SELECT user, host FROM mysql.user
WHERE user = 'joao';
Autenticação: o joao já entra, mas ainda não pode nada.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTENTICAÇÃO — POLÍTICA DE SENHA SISTEMAS
ALTER USER 'joao'@'localhost'
PASSWORD EXPIRE INTERVAL 90 DAY;
ALTER USER 'joao'@'localhost'
FAILED_LOGIN_ATTEMPTS 3
PASSWORD_LOCK_TIME 1;
Senha vence em 90 dias; 3 erros bloqueiam a conta por 1 dia.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTENTICAÇÃO x AUTORIZAÇÃO SISTEMAS
-- no terminal:
mysql -u joao -p
-- já dentro do MySQL, como joao:
SELECT CURRENT_USER();
SHOW DATABASES;
-- ERRO: SELECT command denied
SELECT * FROM escola.aluno;
Autenticado sim (entrou); autorizado não (sem privilégios).
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTORIZAÇÃO — CONCEDENDO SISTEMAS
GRANT SELECT, INSERT ON escola.aluno
TO 'joao'@'localhost';
SHOW GRANTS FOR 'joao'@'localhost';
Privilégios no nível da tabela escola.aluno.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTORIZAÇÃO — TESTANDO O ACESSO SISTEMAS
-- conectado como joao:
SELECT id_aluno, nome FROM escola.aluno;
INSERT INTO escola.aluno
(cpf, nome, idade, email) VALUES
('44444444444', 'Duda', 16, 'd@e.com');
-- ERRO: UPDATE command denied
UPDATE escola.aluno SET nome = 'X'
WHERE id_aluno = 1;
SELECT e INSERT funcionam; UPDATE não foi concedido.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUTORIZAÇÃO — REVOGANDO SISTEMAS
-- de volta ao administrador (root):
REVOKE INSERT ON escola.aluno
FROM 'joao'@'localhost';
SHOW GRANTS FOR 'joao'@'localhost';
Agora o joao só consulta a tabela aluno.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
GRANULARIDADE — POR COLUNA SISTEMAS
CREATE USER 'maria'@'localhost'
IDENTIFIED BY 'Maria#2025';
GRANT SELECT (id_aluno, nome, email)
ON escola.aluno
TO 'maria'@'localhost';
Privilégio só nas colunas id_aluno, nome e email.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
GRANULARIDADE — TESTANDO SISTEMAS
-- conectada como maria:
SELECT nome, email FROM escola.aluno;
-- ERRO: sem permissão na coluna cpf
SELECT cpf FROM escola.aluno;
O CPF fica protegido mesmo com acesso à tabela.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DAC — DONO CONCEDE ACESSO SISTEMAS
CREATE USER 'prof_ana'@'localhost'
IDENTIFIED BY 'ProfAna#2025';
CREATE USER 'prof_bia'@'localhost'
IDENTIFIED BY 'ProfBia#2025';
GRANT SELECT, INSERT ON escola.aluno
TO 'prof_ana'@'localhost'
WITH GRANT OPTION;
WITH GRANT OPTION permite à ana repassar o acesso.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DAC — REPASSE PELA PROFESSORA SISTEMAS
-- conectada como prof_ana:
GRANT SELECT ON escola.aluno
TO 'prof_bia'@'localhost';
-- ERRO: a ana não possui DELETE
GRANT DELETE ON escola.aluno
TO 'prof_bia'@'localhost';
Só se repassa o que se possui.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DAC — RETIRANDO O REPASSE SISTEMAS
-- como administrador (root):
REVOKE GRANT OPTION ON escola.aluno
FROM 'prof_ana'@'localhost';
SHOW GRANTS FOR 'prof_ana'@'localhost';
SHOW GRANTS FOR 'prof_bia'@'localhost';
A ana perde o repasse; a bia mantém o SELECT recebido.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
MAC — DOCUMENTOS CLASSIFICADOS SISTEMAS
CREATE TABLE documento (
id INT PRIMARY KEY AUTO_INCREMENT,
titulo VARCHAR(80) NOT NULL,
nivel TINYINT NOT NULL
CHECK (nivel BETWEEN 1 AND 3)
);
INSERT INTO documento (titulo, nivel)
VALUES ('Cardápio', 1), ('Notas', 2),
('Ata secreta', 3);
Níveis: 1 público, 2 confidencial, 3 sigiloso.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
MAC — HABILITAÇÃO DOS USUÁRIOS SISTEMAS
CREATE TABLE habilitacao (
usuario VARCHAR(30) PRIMARY KEY,
nivel TINYINT NOT NULL
);
INSERT INTO habilitacao VALUES
('prof_ana', 2), ('prof_bia', 1);
A política central define o nível de cada usuário.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
MAC — VIEW QUE APLICA A POLÍTICA SISTEMAS
CREATE VIEW documento_permitido AS
SELECT d.id, d.titulo, d.nivel
FROM documento d
JOIN habilitacao h
ON h.usuario =
SUBSTRING_INDEX(USER(), '@', 1)
WHERE d.nivel <= h.nivel;
Cada usuário só enxerga níveis até a sua habilitação.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
MAC — TESTANDO A POLÍTICA SISTEMAS
GRANT SELECT ON escola.documento_permitido
TO 'prof_ana'@'localhost';
GRANT SELECT ON escola.documento_permitido
TO 'prof_bia'@'localhost';
-- ana vê níveis 1 e 2; bia vê só o 1
SELECT * FROM escola.documento_permitido;
O MySQL não tem MAC nativo: aqui foi simulado com view.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
RBAC — CRIANDO PAPÉIS SISTEMAS
CREATE ROLE 'r_secretaria',
'r_professor',
'r_aluno';
Papéis (roles) agrupam privilégios e ainda não têm dono.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
RBAC — PRIVILÉGIOS DOS PAPÉIS SISTEMAS
GRANT SELECT, INSERT, UPDATE, DELETE
ON escola.aluno TO 'r_secretaria';
GRANT SELECT ON escola.aluno
TO 'r_professor';
GRANT SELECT ON escola.turma
TO 'r_aluno';
Os privilégios vão para o papel, não para a pessoa.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
RBAC — USUÁRIO RECEBE O PAPEL SISTEMAS
CREATE USER 'sec_carla'@'localhost'
IDENTIFIED BY 'Carla#2025';
GRANT 'r_secretaria'
TO 'sec_carla'@'localhost';
SET DEFAULT ROLE 'r_secretaria'
TO 'sec_carla'@'localhost';
A Carla herda tudo o que a secretaria pode fazer.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
RBAC — VERIFICANDO E REVOGANDO SISTEMAS
SHOW GRANTS FOR 'sec_carla'@'localhost'
USING 'r_secretaria';
-- a Carla deixa a secretaria:
REVOKE 'r_secretaria'
FROM 'sec_carla'@'localhost';
Trocar de função é mudar o papel, não refazer GRANTs.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
QUANDO USAR CADA MODELO? SISTEMAS
| MODELO | QUEM DECIDE      | PRATICAMOS          |
| ------ | ---------------- | ------------------- |
| DAC    | O dono do dado   | WITH GRANT OPTION   |
| MAC    | Política central | view por nível      |
| RBAC   | O papel (role)   | CREATE ROLE + GRANT |
Empresas e escolas costumam adotar RBAC.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PREPARANDO O DESAFIO SISTEMAS
CREATE TABLE nota (
id_aluno INT, id_turma INT,
valor DECIMAL(4,1)
CHECK (valor BETWEEN 0 AND 10),
PRIMARY KEY (id_aluno, id_turma)
);
Tabelas do desafio: aluno, turma, matricula e nota.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DESAFIO DO BLOCO 2 SISTEMAS
UNIDADE 1 — SEGURANÇA EM BANCO DE DADOS
Exercício orientado
Defina os privilégios de Secretaria, Professor e Aluno
2
e escreva os GRANT dos três papéis, usando aluno,
turma, matricula e nota.
Resolução individual, com correção coletiva ao final.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CORREÇÃO — SECRETARIA SISTEMAS
-- Secretaria: cadastro completo
GRANT SELECT, INSERT, UPDATE, DELETE
ON escola.aluno TO 'r_secretaria';
GRANT SELECT, INSERT, UPDATE, DELETE
ON escola.matricula TO 'r_secretaria';
GRANT SELECT, INSERT, UPDATE
ON escola.turma TO 'r_secretaria';
A secretaria cadastra alunos, turmas e matrículas.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CORREÇÃO — PROFESSOR E ALUNO SISTEMAS
-- Professor: consulta e lança notas
GRANT SELECT ON escola.aluno
TO 'r_professor';
GRANT SELECT, INSERT, UPDATE
ON escola.nota TO 'r_professor';
-- Aluno: só consulta
GRANT SELECT ON escola.nota
TO 'r_aluno';
Na prática, o aluno só deve ver as próprias notas.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
BLOCO 3 — PRIVACIDADE SISTEMAS
~40 minutos, LGPD na prática:
INVENTÁRIO E CRIPTOGRAFIA
1
~10 min — dados sensíveis e AES
PSEUDONIMIZAÇÃO E ANONIMIZAÇÃO
2
~8 min — mapa e relatório
AUDITORIA E DIREITOS
3
~12 min — trigger e comandos da LGPD
DESAFIO FINAL
4
~10 min — mascaramento de dados
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
LGPD — TABELA DE TITULARES SISTEMAS
CREATE TABLE titular (
id_titular INT PRIMARY KEY AUTO_INCREMENT,
nome VARCHAR(80) NOT NULL,
cpf CHAR(11) UNIQUE NOT NULL,
email VARCHAR(100) NOT NULL,
cidade VARCHAR(40),
saude VARBINARY(255),
consentimento TINYINT(1) DEFAULT 0
);
saude é dado sensível: será guardado criptografado.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
INVENTÁRIO DE DADOS SISTEMAS
CREATE TABLE inventario_dados (
coluna VARCHAR(30) PRIMARY KEY,
tipo VARCHAR(10) NOT NULL
CHECK (tipo IN ('PESSOAL','SENSIVEL')),
finalidade VARCHAR(60) NOT NULL
);
Princípio da finalidade: cada dado tem um motivo registrado.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
INVENTÁRIO — PREENCHENDO SISTEMAS
INSERT INTO inventario_dados VALUES
('nome', 'PESSOAL', 'identificar'),
('cpf', 'PESSOAL', 'identificar'),
('email', 'PESSOAL', 'contato'),
('saude', 'SENSIVEL', 'atendimento');
SELECT * FROM inventario_dados;
Dados pessoais x sensíveis, com finalidade declarada.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CRIPTOGRAFIA — GRAVANDO SISTEMAS
SET @k = SHA2('chave-mestra', 256);
INSERT INTO titular (nome, cpf, email,
cidade, saude, consentimento) VALUES
('Ana','11111111111','ana@mail.com',
'Rio Verde',AES_ENCRYPT('asma',@k),1),
('Bia','22222222222','bia@mail.com',
'Jataí',AES_ENCRYPT('rinite',@k),0);
A chave (@k) vive fora do banco; aqui é só demonstração.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
CRIPTOGRAFIA — LENDO OS DADOS SISTEMAS
-- sem a chave: conteúdo ilegível
SELECT nome, saude FROM titular;
-- com a chave: conteúdo legível
SELECT nome,
CAST(AES_DECRYPT(saude, @k) AS CHAR)
AS saude
FROM titular;
Só quem tem a chave lê o dado sensível.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PSEUDONIMIZAÇÃO — O MAPA SISTEMAS
CREATE TABLE mapa_pseudonimo (
codigo CHAR(10) PRIMARY KEY,
id_titular INT UNIQUE NOT NULL,
FOREIGN KEY (id_titular)
REFERENCES titular(id_titular)
ON DELETE CASCADE
);
O mapa fica separado, com acesso restrito ao controlador.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PSEUDONIMIZAÇÃO — GERANDO CÓDIGOS SISTEMAS
SET @sal = '#segredo';
INSERT INTO mapa_pseudonimo
SELECT LEFT(SHA2(CONCAT(cpf,@sal),256),10),
id_titular
FROM titular;
SELECT * FROM mapa_pseudonimo;
Código derivado do CPF + segredo: só reverte quem tem o mapa.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
View em SQL SISTEMAS
Uma view é uma tabela virtual definida por uma consulta
armazenada no banco. Ela não guarda dados próprios
SELECT
(exceto as materializadas): ao ser consultada,
executa a query original.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
View em SQL SISTEMAS
Uso e características
Para que é usada
Simplificar consultas complexas (joins, agregações)
Restringir acesso a colunas/linhas (segurança)
Padronizar regras de negócio e relatórios
Manter compatibilidade quando a estrutura das tabelas muda
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
View em SQL SISTEMAS
Características
Criada com
CREATE VIEW nome AS SELECT ...
Consultada como uma tabela comum
Reflete sempre os dados atuais das tabelas base
Pode ser atualizável (se simples) ou somente leitura
Existe a versão materializada, que armazena o resultado
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
View em SQL SISTEMAS
Pontos fortes
Reuso de lógica: escreve uma vez, usa em vários lugares
Segurança: expõe só o necessário (ex.: oculta salário)
Abstração: o usuário não precisa conhecer o esquema
Facilita manutenção e organização do código SQL
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SISTEMAS
| View | em SQL |     |     |     |     |
| ---- | ------ | --- | --- | --- | --- |
Pontos fracos
| Pode prejudicar o desempenho (views |     |     |     | aninhadas,  |     |
| ----------------------------------- | --- | --- | --- | ----------- | --- |
queries pesadas)
| Atualização ( |     | /   | ) limitada em views |     | complexas |
| ------------- | --- | --- | ------------------- | --- | --------- |
INSERT UPDATE
Dependência das tabelas base: alterações podem quebrá-las
| Views | materializadas exigem refresh |     |     | e ocupam espaço |     |
| ----- | ----------------------------- | --- | --- | --------------- | --- |
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PSEUDONIMIZAÇÃO — VISÃO ANALÍTICA SISTEMAS
CREATE VIEW analise_pseudonimizada AS
SELECT m.codigo, t.cidade
FROM titular t
JOIN mapa_pseudonimo m USING (id_titular);
SELECT * FROM analise_pseudonimizada;
O analista vê códigos e cidade, sem nome nem CPF.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
ANONIMIZAÇÃO — SEM VOLTA SISTEMAS
CREATE TABLE relatorio_anonimo AS
SELECT cidade, COUNT(*) AS total
FROM titular
GROUP BY cidade;
SELECT * FROM relatorio_anonimo;
Sem nome, CPF ou código: não há como voltar ao titular.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUDITORIA — TABELA DE LOG SISTEMAS
CREATE TABLE log_auditoria (
id INT PRIMARY KEY AUTO_INCREMENT,
usuario VARCHAR(100) NOT NULL,
acao VARCHAR(10) NOT NULL,
id_titular INT,
data_hora DATETIME NOT NULL
DEFAULT CURRENT_TIMESTAMP
);
Registra quem fez o quê e quando.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUDITORIA — TRIGGER (1/2) SISTEMAS
DELIMITER $$
CREATE TRIGGER audit_titular_upd
AFTER UPDATE ON titular
FOR EACH ROW
BEGIN
O trigger dispara depois de cada UPDATE em titular.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUDITORIA — TRIGGER (2/2) SISTEMAS
INSERT INTO log_auditoria
(usuario, acao, id_titular)
VALUES
(CURRENT_USER(), 'UPDATE',
NEW.id_titular);
END$$
DELIMITER ;
NEW.id_titular é o id da linha alterada.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
AUDITORIA — TESTANDO SISTEMAS
UPDATE titular
SET email = 'ana.souza@mail.com'
WHERE id_titular = 1;
SELECT usuario, acao, id_titular, data_hora
FROM log_auditoria;
Triggers não registram SELECT: use um plugin de auditoria.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DIREITOS — CONFIRMAÇÃO E ACESSO SISTEMAS
-- 1) Confirmação do tratamento
SELECT COUNT(*) AS existe
FROM titular WHERE cpf = '11111111111';
-- 2) Acesso aos dados
SELECT nome, cpf, email, cidade
FROM titular WHERE cpf = '11111111111';
Direitos do titular: confirmar e acessar os dados.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DIREITOS — CORREÇÃO E PORTABILIDADE SISTEMAS
-- 3) Correção
UPDATE titular SET cidade = 'Jataí'
WHERE cpf = '11111111111';
-- 4) Portabilidade (formato JSON)
SELECT JSON_OBJECT('nome', nome,
'email', email, 'cidade', cidade)
FROM titular WHERE cpf = '11111111111';
Portabilidade: entregar os dados em formato estruturado.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DIREITOS — ELIMINAÇÃO SISTEMAS
-- 5) Eliminação dos dados
DELETE FROM titular
WHERE cpf = '22222222222';
SELECT * FROM mapa_pseudonimo;
SELECT * FROM titular;
O DELETE CASCADE também apagou o pseudônimo da Bia.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
DESAFIO FINAL SISTEMAS
UNIDADE 1 — PRIVACIDADE DE DADOS E LGPD
Exercício orientado
Escolha um sistema (escola ou e-commerce): 1)
3
dados pessoais e sensíveis; 2) titular, controlador e
operador; 3) uma medida técnica em SQL.
Resolução individual, com correção coletiva ao final.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
EXEMPLO — MASCARAMENTO DE DADOS SISTEMAS
CREATE VIEW titular_mascarado AS
SELECT nome,
CONCAT(LEFT(email, 2), '***@',
SUBSTRING_INDEX(email, '@', -1)) AS email,
CONCAT('*******', RIGHT(cpf, 4)) AS cpf
FROM titular;
SELECT * FROM titular_mascarado;
Mascaramento aplica o princípio da necessidade.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
SÍNTESE DA AULA SISTEMAS
INTEGRIDADE
1
CHECK, PK, FK, UNIQUE e TRIGGER
SEGURANÇA
2
CREATE USER, GRANT, REVOKE e ROLE
PRIVACIDADE E LGPD
3
AES, hash, auditoria e direitos
Refaça os scripts do Word e os desafios em casa.
www.unirv.edu.br

BANCO DE DADOS II
PROF. JÚLIO
ANALISTA DE
PRÓXIMO ENCONTRO SISTEMAS
• Unidade 2: Elaboração e Implementação de um Projeto de
Sistemas de Banco de Dados
• Fases de um projeto de banco de dados e levantamento de
requisitos
Lembrete: 20h da carga horária são atividades direcionadas —
exercícios de SQL resolvidos fora da aula.
www.unirv.edu.br