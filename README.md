# CheatSheets

___

## Entity Framework

### Data Annotations

#### Tables

    [Table(“tbl_Course”, Schema = “catalog”)]
    public class Course
    {
    }

#### Columns

   [Column(“sName”, TypeName = “varchar”)]
   [Required]
   [MaxLength(255)]
   public string Name { get; set; }

#### Primary Keys

    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.None)]
    public class Isbn { get; set; }
