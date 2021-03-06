---
permalink: approach-database-first
---

In the previous article, we have seen the migrations to create a database from a DbContext and classes. It is also possible to reverse engineer an existing database into a DbContext and classes, and it is known as **Database First** approach. 

 - We have a simple database created in the previous article, and it contains two tables, `Order` and `OrderDetail`. 
 - To create a DbContext and classes from the existing database, we will run the `Scaffold-DbContext` command in Package Manager console. 

Let's create a new empty project and add all the required packages for EF core which is explained in the [Code First](/approach-code-first) approach. 

<img src="{{ site.github.url }}/images/database-first1.png">

Now we need to run the `scaffold-dbcontext` command with just the required parameters.

`PM> Scaffold-DbContext -Provider Microsoft.EntityFrameworkCore.SqlServer -Connection "Data Source=(localdb)\ProjectsV13;Initial Catalog=StoreDB;"`

The provider and connection parameters are required, the provider is Microsoft.EntityFrameworkCore.SqlServer and the connection is the connection string for our legacy database.

When you run this command, a bunch of new files is added in the Solution Explorer. 

<img src="{{ site.github.url }}/images/database-first2.png">

Now let's look at the new DbContext class. 

{% include template-example.html %} 
{% highlight csharp %}

public partial class StoreDBContext : DbContext
{
    public virtual DbSet<OrderDetails> OrderDetails { get; set; }
    public virtual DbSet<Orders> Orders { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
#warning To protect potentially sensitive information in your connection string, you should move it out of source code. See http://go.microsoft.com/fwlink/?LinkId=723263 for guidance on storing connection strings.
            optionsBuilder.UseSqlServer(@"Data Source=(localdb)\ProjectsV13;Initial Catalog=StoreDB;");
        }
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<OrderDetails>(entity =>
        {
            entity.HasKey(e => e.OrderDetailId);

            entity.HasIndex(e => e.OrderId);

            entity.Property(e => e.OrderDetailId).HasColumnName("OrderDetailID");

            entity.Property(e => e.OrderId).HasColumnName("OrderID");

            entity.Property(e => e.ProductId).HasColumnName("ProductID");

            entity.HasOne(d => d.Order)
                .WithMany(p => p.OrderDetails)
                .HasForeignKey(d => d.OrderId);
        });

        modelBuilder.Entity<Orders>(entity =>
        {
            entity.HasKey(e => e.OrderId);

            entity.Property(e => e.OrderId).HasColumnName("OrderID");

            entity.Property(e => e.CustomerId).HasColumnName("CustomerID");

            entity.Property(e => e.EmployeeId).HasColumnName("EmployeeID");
        });
    }
}

{% endhighlight %}

 - You can see that DbSet properties are created for every one of the Entity classes. 
 - It also put the connection string directly in the OnConfiguring method with a note that you might not want it there for security reasons.
 - There are also a bunch of explicit fluent mappings for the indexes and relationship etc.
