# .NET | Entity Framework - Mappings de Entidades

Ao utilizar o Entity Framework, para conseguirmos relacionar uma tabela de banco de dados à uma classe, precisamos implementar este relacionamento.<br/>
Para isso, realizamos o 'mapping' da classe (conhecida mais especificamente como ENTIDADE) com a tabela através de algumas configurações do Entity Framework.<br/>
O mapeamento é simples, mas ao mesmo tempo bastante completo em quesitos de configurações possíveis.<br/>
Com ele temos, dentro de várias outras, a possibilidade de: <br/>
- Realizar relacionamentos entre entidades
- Especificar o tipo da coluna
- Nome da tabela referente aquela entidade
- Nome da coluna
- Se a coluna é obrigatória ou não
- Index
- Conversões antes de salvar e antes de buscar os dados

### Ok, mas como isso é feito?
Suponhamos que temos as entidades a seguir:

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

public class Produto
{
    public Guid Id { get;  set; }
    public DateTime DataCriacao { get;  set; }
    public string Nome { get;  set; }
    public decimal Preco { get;  set; }
    public string Imagem { get;  set; }
}
```
Com isso, vamos criar os mappings para cada entidade
``` c#
public class CompraMapping : IEntityTypeConfiguration<Compra>
{
    public void Configure(EntityTypeBuilder<Compra> builder)
    {
        //NOME DA TABELA
        builder.ToTable("COMPRA");
        //CHAVE PRIMÁRIA
        builder.HasKey(x => x.Id);
        //NOME DA COLUNA E NÃO PODE SER NULA
        builder.Property(x => x.DataCriacao).HasColumnName("DATACRIACAO").IsRequired();
        //NOME DA COLUNA E NÃO PODE SER NULA
        builder.Property(x => x.IdUsuario).HasColumnName("IDUSUARIO").IsRequired();
        //NOME DA COLUNA E NÃO PODE SER NULA
        builder.Property(x => x.ValorTotal).HasColumnName("VALORTOTAL").IsRequired();
        //RELACIONAMENTO ENTRE AS ENTIDADES E INSERINDO UMA CONSTRAINT DE EXCLUSÃO EM CASCATA
        builder.HasMany(x => x.Produtos).WithOne(x => x.Compra).HasForeignKey(x => x.IdCompra).OnDelete(DeleteBehavior.Cascade);
        //RELACIONAMENTO ENTRE AS ENTIDADES E INSERINDO UMA CONSTRAINT DE EXCLUSÃO EM CASCATA
        builder.HasOne(x => x.Usuario).WithMany().HasForeignKey(x => x.IdUsuario).OnDelete(DeleteBehavior.Cascade);
    }
}

public class CompraProdutoMapping : IEntityTypeConfiguration<CompraProduto>
{
    public void Configure(EntityTypeBuilder<CompraProduto> builder)
    {
        builder.ToTable("COMPRAPRODUTO");
        builder.HasKey(x => x.Id);
        builder.Property(x => x.DataCriacao).HasColumnName("DATACRIACAO").IsRequired();
        builder.Property(x => x.IdProduto).HasColumnName("IDPRODUTO").IsRequired();
        builder.Property(x => x.IdCompra).HasColumnName("IDCOMPRA").IsRequired();
        builder.Property(x => x.ValorTotal).HasColumnName("VALORTOTAL").IsRequired();
        builder.Property(x => x.Quantidade).HasColumnName("QUANTIDADE").IsRequired();
        builder.HasOne(x => x.Produto).WithMany().HasForeignKey(x => x.IdProduto).OnDelete(DeleteBehavior.Cascade);
        builder.HasOne(x => x.Compra).WithMany().HasForeignKey(x => x.IdCompra).OnDelete(DeleteBehavior.Cascade);
    }
}

public class ProdutoMapping : IEntityTypeConfiguration<Produto>
{
    public void Configure(EntityTypeBuilder<Produto> builder)
    {
        builder.ToTable("PRODUTO");
        builder.HasKey(x => x.Id);
        builder.Property(x => x.DataCriacao).HasColumnName("DARACRIACAO").IsRequired();
        //NOME DA COLUNA, NÃO PODE SER NULA E SETADO O TIPO ESPECÍFICO COMO VARCHAR(100) 
        builder.Property(x => x.Nome).HasColumnName("NOME").HasColumnType("VARCHAR(100)").IsRequired();
        builder.Property(x => x.Imagem).HasColumnName("IMAGEM").IsRequired();
        builder.Property(x => x.Preco).HasColumnName("PRECO").IsRequired();
    }
}

public class UsuarioMapping : IEntityTypeConfiguration<Usuario>
{
    public void Configure(EntityTypeBuilder<Usuario> builder)
    {
        builder.ToTable("USUARIO");
        builder.HasKey(x => x.Id);
        builder.Property(x => x.DataCriacao).HasColumnName("DARACRIACAO").IsRequired();
        builder.Property(x => x.Nome).HasColumnName("NOME").HasColumnType("VARCHAR(100)").IsRequired();
        builder.Property(x => x.Telefone).HasColumnName("TELEFONE").HasColumnType("VARCHAR(15)").IsRequired();
        builder.Property(x => x.Email).HasColumnName("EMAIL").HasColumnType("VARCHAR(100)").IsRequired();
    }
}
```
Certo, com isso temos o nosso mapeamento. Porém, temos que inserir estas configurações no contexto do Entity para que o mesmo aplique estas configurações e também para termos acesso às nossas tabelas mais facilmente.
Então vamos criar o nosso contexto:

``` c#
 public class ContextoEntity : DbContext
    {
        public ContextoEntity(DbContextOptions<ContextoEntity> options) : base(options)
        {
        }


        public DbSet<Usuario> Usuario { get; set; }
        public DbSet<Compra> Compra { get; set; }
        public DbSet<CompraProduto> CompraProduto { get; set; }
        public DbSet<Produto> Produto { get; set; }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.ApplyConfigurationsFromAssembly(GetType().Assembly);
            base.OnModelCreating(builder);
        }
    }
```
##### Explicações:
- DbSet<Type> -> Ajuda a acessarmos mais rapidamente as nossas tabelas. Podemos ter um para cada tabela.
- builder.ApplyConfigurationsFromAssembly(GetType().Assembly) -> Executa, de maneira mais automatizada, as configurações de mapeamento do Assembly corrente, evitando que tenhamos que colocar manualmente cada mapeamento para ser aplicado.
    
#### Pacotes necessários:
- microsoft.entityframeworkcore
- microsoft.entityframeworkcore.design
- microsoft.entityframeworkcore.relational
- microsoft.entityframeworkcore.tools
