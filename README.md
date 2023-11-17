# Biblioteca
## o intuito da atividade e mostrar como funciona a criação de um sistema de armazenagem de uma bliblioteca, nesse sistema poderemos ver os livros, autores e editoras, comom tambem poderemos ver os clientes e seus emprestimos
# *___________________________________________________________________________*
![image](https://github.com/GabrielChagasAlves/Biblioteca/assets/125607847/e8dd5d84-6109-439f-ae34-fab9b17e058a)

# *___________________________________________________________________________*
### Autores: Registre informações sobre os autores, como nome, data de nascimento e nacionalidade.
![image](https://github.com/GabrielChagasAlves/Biblioteca/assets/125607847/0c04575a-4f08-4d58-b2f4-ebfdee850d6a)

# *___________________________________________________________________________*
### Editoras: Mantenha detalhes sobre as editoras, como nome e endereço.
![image](https://github.com/GabrielChagasAlves/Biblioteca/assets/125607847/8eb4da0e-a849-4602-909a-8ab12beafc6e)

# *___________________________________________________________________________*
### Mantenha os dados dos clientes como nome, cpf, data de nascimento, endereço
![image](https://github.com/GabrielChagasAlves/Biblioteca/assets/125607847/c2088b21-6907-4f7c-9445-a66fdd089d67)

# *___________________________________________________________________________*
### Empréstimos: Controle os empréstimos de livros, incluindo a data de empréstimo e de devolução, bem como o status do empréstimo (pendente, devolvido, atrasado).
![image](https://github.com/GabrielChagasAlves/Biblioteca/assets/125607847/4a952c7e-19c1-48ce-8eec-745c70fad832)

# *___________________________________________________________________________*
### Crie uma stored procedure para registrar um novo empréstimo, verificando a disponibilidade do livro e atualizando o estoque.
```SQL
DELIMITER //

CREATE PROCEDURE RegistrarEmprestimo(
    IN livro_id INT UNSIGNED,
    IN cliente_id INT UNSIGNED,
    IN data_emprest DATE,
    IN data_devo DATE
)
BEGIN
    DECLARE disponibilidade INT;

    -- Check book availability
    SELECT estoque INTO disponibilidade FROM Livro WHERE id_livro = livro_id;

    -- If the book is available, proceed with the loan registration
    IF disponibilidade > 0 THEN
        -- Update book stock
        UPDATE Livro SET estoque = estoque - 1 WHERE id_livro = livro_id;

        -- Register the loan
        INSERT INTO Emprestimos (data_emprest, data_devo, status_emprest, Cliente_id_Cliente)
        VALUES (data_emprest, data_devo, 'Pendente', cliente_id);

        SELECT 'Empréstimo registrado com sucesso.' AS mensagem;

    ELSE
        -- If the book is not available, display a message
        SELECT 'Livro não disponível no estoque. Empréstimo não registrado.' AS mensagem;
    END IF;
END //

DELIMITER ;

```

# *___________________________________________________________________________*
### Crie outra stored procedure para recuperar a lista de livros emprestados por um cliente específico.
```SQL
DELIMITER //

CREATE PROCEDURE LivrosEmprestadosPorCliente(
    IN cliente_id INT UNSIGNED
)
BEGIN
    -- Retrieve the list of borrowed books for the specified client
    SELECT Livro.titulo, Livro.ISBN, Emprestimos.data_emprest, Emprestimos.data_devo
    FROM Livro
    JOIN Emprestimos ON Livro.id_livro = Emprestimos.Emprestimos_id_Emprestimos
    WHERE Emprestimos.Cliente_id_Cliente = cliente_id;

END //

DELIMITER ;

```

# *___________________________________________________________________________*
### Implemente uma stored procedure que calcule multas para empréstimos atrasados.
```SQL
DELIMITER //

CREATE PROCEDURE CalcularMultas()
BEGIN
    DECLARE multa_diaria DECIMAL(10, 2);
    
    -- Set the daily fine amount (adjust as needed)
    SET multa_diaria = 5.00;

    -- Update overdue loans with calculated fines
    UPDATE Emprestimos
    SET status_emprest = 'Atrasado',
        multa = DATEDIFF(CURRENT_DATE, data_devo) * multa_diaria
    WHERE data_devo < CURRENT_DATE AND status_emprest = 'Pendente';

    SELECT 'Multas calculadas e atualizadas com sucesso.' AS mensagem;
END //

DELIMITER ;

```

# *___________________________________________________________________________*
### Crie uma view que mostre os livros disponíveis para empréstimo, excluindo aqueles que já foram emprestados.
```SQL
CREATE VIEW LivrosDisponiveis AS
SELECT Livro.id_livro, livro.titulo, Livro.ISBN, Livro.AnoDePublicação, livro.estoque
FROM Livro
LEFT JOIN Emprestimos ON Livro.id_livro = Emprestimos.Emprestimos_id_Emprestimos
WHERE (Emprestimos.id_Emprestimos IS NULL) OR (Emprestimos.status_emprest = 'Devolvido');

```

# *___________________________________________________________________________*
### Implemente uma view que forneça uma lista de todos os empréstimos atuais, incluindo os detalhes dos livros emprestados e dos clientes.
```SQL
CREATE VIEW ListaEmprestimos AS
SELECT
    Emprestimos.id_Emprestimos,
    Emprestimos.data_emprest,
    Emprestimos.data_devo,
    Emprestimos.status_emprest,
    Cliente.id_Cliente,
    Cliente.nome_cliente,
    Cliente.cpf,
    Cliente.data_de_nasc,
    Cliente.endereco,
    Livro.id_livro,
    Livro.titulo AS titulo_livro,
    Livro.ISBN,
    Livro.AnoDePublicacao
FROM
    Emprestimos
JOIN Cliente ON Emprestimos.Cliente_id_Cliente = Cliente.id_Cliente
JOIN Livro ON Emprestimos.Emprestimos_id_Emprestimos = Livro.id_livro;

```
