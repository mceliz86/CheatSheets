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


## LINQ

### Restriction
    from c in context.Courses
    where c.Level == 1
    select c;

### Ordering
    from c in context.Courses
    where c.Level == 1
    orderby c.Name, c.Price descending
    select c;

### Projection
    from c in context.Courses
    select new { Course = c.Name, AuthorName = c.Author.Name };

### Grouping
    from c in context.Courses
    group c by c.Level into g
    select g;

    from c in context.Courses
    group c by c.Level into g
    select new { Level = g.Key, Courses = g };
    
### Inner Join
Use when there is no relationship between your entities and you need to link them based on a key 

    from a in context.Authors
    join c in context.Courses on a.Id equals c.AuthorId
    select new { Course = c.Name, Author = a.Name };

### Group Join
Useful when you need to group objects by a property and count the number of objects in each group. In SQL we do this with LEFT JOIN, COUNT(*) and GROUP BY. In LINQ, we use group join.

    from a in context.Authors
    join c in context.Courses on a.Id equals c.AuthorId into g
    select new { Author = a.Name, Courses = c.Count() };

### Cross Join
To get full combinations of all objects on the left and the ones on the right.

    from a in context.Authors
    from c in context.Courses
    select new { Author = a.Name, Course = c.Name };
    

## Extension methods

### Restriction
    context.Courses.Where(c => c.Level == 1);

### Ordering
    context.Courses
    .OrderBy(c => c.Name) // or OrderByDescending
    .ThenBy(c => c.Level); // or ThenByDescending

### Projection
    context.Courses.Select(c => new
    {
        CourseName = c.Name, AuthorName = c.Author.Name // no join required
    });

    //flattens hierarchical lists
    var tags = context.Courses.SelectMany(c => c.Tags);

### Grouping
    var groups = context.Courses.GroupBy(c => c.Level);
    
### Inner Join
Use when there is no relationship between your entities and you need to link them based on a key.

    var authors = context.Authors.Join(context.Courses,
    a => a.Id, // key on the left side
    c => c.AuthorId, // key on the right side,
    (author, course) => // what to do once matched
    new
    {
        AuthorName = author.Name,
        CourseName = course.Name
    }
    );

### Group Join

Useful when you need to group objects by a property and count the number of objects in each group. In SQL we do this with LEFT JOIN, COUNT(*) and GROUP BY. In LINQ, we use group join.

    var authors = context.Authors.GroupJoin(context.Courses,
    a => a.Id, // key on the left side
    c => c.AuthorId, // key on the right side,
    (author, courses) => // what to do once matched
    new
    {
        AuthorName = author.Name,
        Courses = courses
    }
    );
    
### Cross Join
To get full combinations of all objects on the left and the ones on the right.

    var combos = context.Authors.SelectMany(a => context.Courses,
    (author, course) => new {
        AuthorName = author.Name,
        CourseName = course.Name
    });
    
### Partitioning
To get records in a given page.

    context.Courses.Skip(10).Take(10);

### Element Operators
// throws an exception if no elements found

    context.Courses.First();
    context.Courses.First(c => c.Level == 1);

// returns null if no elements found

    context.Courses.FirstOrDefault();

// not supported by SQL Server

    context.Courses.Last();
    context.Courses.LastOrDefault();
    
    context.Courses.Single(c => c.Id == 1);
    context.Courses.SingleOrDefault(c => c.Id == 1);
    
### Quantifying
    bool allInLevel1 = context.Courses.All(c => c.Level == 1);
    
    bool anyInLevel1 = context.Courses.Any(c => c.Level == 1);

### Aggregating
    int count = context.Courses.Count();
    int count = context.Courses.Count(c => c.Level == 1);
    
    var max = context.Courses.Max(c => c.Price);
    var min = context.Courses.Min(c => c.Price);
    var avg = context.Courses.Average(c => c.Price);
    var sum = context.Courses.Sum(c => c.Price);
