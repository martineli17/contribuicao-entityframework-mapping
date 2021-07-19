# .NET | Entity Framework - Mappings de Entidades
### O que é?
Ao utilizar o Entity Framework, para conseguirmos relacionar uma tabela de banco de dados à uma classe, precisamos implementar este relacionamento.<br/>
Para isso, realizamos o 'mapping' da classe (conhecida mais especificamente como ENTIDADE) com a tabela através de algumas configurações do Entity Framework.<br/>
O maoeamento é simples, mas ao mesmo tempo bastante completo em quesitos de configurações possíveis.<br/>
Com ele, temos, dentro de várias outras, a possibilidade de: <br/>
- Realizar relacionamentos entre entidades
- Especificar o tipo da coluna
- Nome da tabela referente aquela entidade
- Nome da coluna
- Se a coluna é obrigatória ou não
- Index
- Conversões antes de salvar e antes de buscar os dados

### Ok, mas como isso é feito?
Suponhamos que temos três entidades:
- Usuario
- Compra
- CompraProduto
```c#
public class Compra
{
    public Guid Id { get;  set; }
    public DateTime DataCriacao { get;  set; }
    public Guid IdUsuario { get;  set; }
    public decimal ValorTotal { get;  set; }
    public Usuario Usuario { get;  set; }
    public ICollection<CompraProduto> Produtos { get;  set; }
}

public class CompraProduto
{
    public Guid Id { get;  set; }
    public DateTime DataCriacao { get;  set; }
    public Guid IdProduto { get;  set; }
    public Guid IdCompra { get;  set; }
    public int Quantidade { get;  set; }
    public decimal ValorTotal { get;  set; }
    public Compra Compra { get;  set; }
    public Produto Produto { get;  set; }
}

public class Usuario
{
    public Guid Id { get;  set; }
    public DateTime DataCriacao { get;  set; }
    public string Nome { get;  set; }
    public string Email { get;  set; }
    public string Telefone { get;  set; }
}
```
