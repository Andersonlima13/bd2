--- Script, apresentacao




--View de exeplo de uso
--Essa visão fornece uma visão consolidada sobre o monitoramento dos prédios e
--os eventos registrados pelas câmeras. Inclui informações sobre o prédio, funcionário,
--evento e a câmera que registrou o evento.

CREATE VIEW VISAO_MONITORAMENTO_EVENTOS AS
SELECT
P.ENDERECO AS ENDERECO_PREDIO,
F.NOME AS NOME_FUNCIONARIO,

E.DESCRICAO AS DESCRICAO_EVENTO,
E.DATA_DE_EVENTO AS DATA_EVENTO,
CE.ID_CAMERA AS ID_CAMERA
FROM
PREDIO P
JOIN
MONITORAMENTO M ON P.ID_PREDIO = M.ID_PREDIO
JOIN
FUNCIONARIO F ON M.COD_FUNCIONARIO = F.COD_FUNCIONARIO
JOIN
REGISTRO_MONITORAMENTO RM ON RM.ID_PREDIO = P.ID_PREDIO
JOIN
EVENTO E ON RM.ID_EVENTO = E.ID_EVENTO
JOIN
CAMERA_EVENTO CE ON CE.ID_EVENTO = E.ID_EVENTO
WHERE
M.PLANTAO = RM.PLANTAO;


---Função de uso exemplo
--- total de eventos que uma camera registrou
CREATE OR REPLACE FUNCTION total_eventos_por_camera(camera_id INT)
RETURNS INT AS $$
DECLARE
total_eventos INT;
BEGIN
SELECT COUNT(*)
INTO total_eventos
FROM CAMERA_EVENTO
WHERE ID_CAMERA = camera_id;
RETURN total_eventos;
END;
$$ LANGUAGE plpgsql;

select * from total_eventos_por_camera(1)



--Trigger para Inserir Registro de Monitoramento com Evento
--Descrição: Cria automaticamente um registro de monitoramento para um prédio quando
---um evento é registrado.

CREATE OR REPLACE FUNCTION inserir_registro_monitoramento()
RETURNS TRIGGER AS $$
BEGIN
-- Inserir um registro de monitoramento para o prédio onde o evento foi registrado
INSERT INTO REGISTRO_MONITORAMENTO (COD_FUNCIONARIO, ID_PREDIO,
PLANTAO, ID_EVENTO)
SELECT F.COD_FUNCIONARIO, C.ID_PREDIO, NOW(), NEW.ID_EVENTO
FROM CAMERA_EVENTO CE
JOIN CAMERA C ON CE.ID_CAMERA = C.ID_CAMERA
JOIN FUNCIONARIO F ON F.COD_FUNCIONARIO = (SELECT COD_FUNCIONARIO
FROM MONITORAMENTO M WHERE M.ID_PREDIO = C.ID_PREDIO LIMIT 1)
WHERE CE.ID_EVENTO = NEW.ID_EVENTO;
RETURN NEW;
END;

$$ LANGUAGE plpgsql;
CREATE TRIGGER trigger_inserir_registro_monitoramento
AFTER INSERT ON EVENTO
FOR EACH ROW
EXECUTE FUNCTION inserir_registro_monitoramento();

