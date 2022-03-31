# Entity Framework

## Data Annotations

### Tables

    [Table(“tbl_Course”, Schema = “catalog”)]
    public class Course
    {
    }

### Columns

    [Column(“sName”, TypeName = “varchar”)]
    [Required]
    [MaxLength(255)]
    public string Name { get; set; }

### Primary Keys

    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public class Isbn { get; set; }

### Composite Primary Keys

    public class OrderItem
    {
        [Key]
        [Column(Order = 1)]
        public int OrderId { get; set; }
        
        [Key]
        [Column(Order = 2)]
        public int OrderItemId { get; set; }
    }

### Indices

    [Index]
    public int AuthorId { get; set; }

    [Index(IsUnique = true)]
    public string Name { get; set; }
    
### Composite Indices

    [Index(“IX_AuthorStudentsCount”,1)]
    public int AuthorId { get; set; }
    
    [Index(“IX_AuthorStudentsCount”, 2)]
    public int StudentsCount { get; set; }

### Foreign Keys

    public class Course
    {
        [ForeignKey(“Author”)]
        public int AuthorId { get; set; }
        
        public Author Author { get; set; }
    }


## Fluent API

### Basics

    public class PlutoContext : DbContext
    {
        protected override void OnModelCreating(DbModelBuilder modelBuilder)
        {
            modelBuilder .Entity<Course> .Property(t => t.Name) .IsRequired();
        }
    }

### Tables

    Entity<Course>.ToTable(“tbl_Course”, “catalog”);

### Primary Keys

    Entity<Book>.HasKey(t => t.Isbn);
    
    Entity<Book>.Property(t => t.Isbn)
    .HasDatabaseGeneratedOption(DatabaseGeneratedOption.None);

### Composite Primary Keys

    Entity<OrderItem>.HasKey(t => new { t.OrderId, t.OrderId });

### Columns

    Entity<Course>.Property(t => t.Name)
    .HasColumnName(“sName”)
    .HasColumnType(“varchar”)
    .HasColumnOrder(2)
    .IsRequired()
    .HasMaxLength(255);

### One-to-many Relationship
    
    Entity<Author>
    .HasMany(a => a.Courses)
    .WithRequired(c => c.Author)
    .HasForeignKey(c => c.AuthorId);

### Many-to-many Relationship

    Entity<Course>
    .HasMany(c => c.Tags)
    .WithMany(t => t.Courses)
    .Map(m =>
    {
        m.ToTable(“CourseTag”);
        m.MapLeftKey(“CourseId”);
        m.MapRightKey(“TagId”);
    });
    
### One-to-zero/one Relationship

    Entity<Course>
    .HasOptional(c => c.Caption)
    .WithRequired(c => c.Course);

### One-to-one Relationship

    Entity<Course>
    .HasRequired(c => c.Cover)
    .WithRequiredPrincipal(c => c.Course);

    Entity<Cover>
    .HasRequired(c => c.Course)
    .WithRequiredDependent(c => c.Cover);
