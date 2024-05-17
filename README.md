# Criando-Transa-es-Executando-Backup-e-Recovery-de-Banco-de-Dados

-- PARTE 1 – TRANSAÇÕES
SET autocommit = 0;
BEGIN;

-- Executando operações na base de dados
INSERT INTO tabela1 (coluna1, coluna2) VALUES ('valor1', 'valor2');
UPDATE tabela2 SET coluna3 = 'novo_valor' WHERE coluna4 = 'valor_antigo';
DELETE FROM tabela3 WHERE coluna5 = 'valor_excluir';

-- Finalizando a transação
COMMIT;

-- PARTE 2 - TRANSAÇÃO COM PROCEDURE
DELIMITER $$
CREATE PROCEDURE exemplo_transacao()
BEGIN
    DECLARE erro_ocorrido BOOLEAN DEFAULT FALSE;
    DECLARE contador INT DEFAULT 0;
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1 @sqlstate = RETURNED_SQLSTATE, @errno = MYSQL_ERRNO, @text = MESSAGE_TEXT;
        SET erro_ocorrido = TRUE;
        ROLLBACK;
        SELECT @text;
    END;

    START TRANSACTION;
    SAVEPOINT inicio_transacao;

    -- Executando operações na base de dados
    INSERT INTO tabela1 (coluna1, coluna2) VALUES ('valor1', 'valor2');
    UPDATE tabela2 SET coluna3 = 'novo_valor' WHERE coluna4 = 'valor_antigo';

    -- Simulando um erro
    DELETE FROM tabela3 WHERE coluna5 = 'valor_invalido';

    IF erro_ocorrido THEN
        ROLLBACK TO SAVEPOINT inicio_transacao;
    ELSE
        COMMIT;
    END IF;
END$$
DELIMITER ;

# PARTE 3 – BACKUP E RECOVERY

# Criando backup do banco de dados e-commerce
mysqldump -u root -p e-commerce > backup_ecommerce.sql

# Criando backup de múltiplos bancos de dados
mysqldump -u root -p --databases db1 db2 db3 > backup_varios_bancos.sql

# Recuperando o banco de dados e-commerce a partir do backup
mysql -u root -p e-commerce < backup_ecommerce.sql
