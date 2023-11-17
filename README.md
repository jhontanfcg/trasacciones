# trasacciones

CREATE DATABASE TransaccionesEjemplo;
-- Selecciona la base de datos recién creada
USE TransaccionesEjemplo;


CREATE TABLE cuentas (
  id INT PRIMARY KEY AUTO_INCREMENT,
  nombre VARCHAR(50),
  saldo DECIMAL(10,2)
);


INSERT INTO cuentas (nombre, saldo) VALUES ('Cuenta A', 1000.00);
INSERT INTO cuentas (nombre, saldo) VALUES ('Cuenta B', 500.00);

DELIMITER //
CREATE PROCEDURE transferirFondos(
    IN p_CuentaOrigen INT,
    IN p_CuentaDestino INT,
    IN p_Monto DECIMAL(10,2)
)
BEGIN
  DECLARE SaldoOrigen DECIMAL(10,2);
  DECLARE SaldoDestino DECIMAL(10,2);

  
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    
    ROLLBACK;
    SELECT 'Error: ', SQLERRM;
  END;

  
  START TRANSACTION;

 
  SELECT saldo INTO SaldoOrigen FROM cuentas WHERE id = p_CuentaOrigen;

  
  IF SaldoOrigen IS NULL OR SaldoOrigen < p_Monto THEN
    
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error: Fondos insuficientes en la cuenta de origen.';
  END IF;

  
  SELECT saldo INTO SaldoDestino FROM cuentas WHERE id = p_CuentaDestino;

  
  IF SaldoDestino IS NULL THEN
   
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Error: La cuenta de destino no existe.';
  END IF;

  
  UPDATE cuentas SET saldo = saldo - p_Monto WHERE id = p_CuentaOrigen;


  UPDATE cuentas SET saldo = saldo + p_Monto WHERE id = p_CuentaDestino;

 
  COMMIT;

  SELECT 'Transferencia completada exitosamente';

END //
DELIMITER ;


CALL transferirFondos(1, 2, 300.00);


SELECT * FROM cuentas
