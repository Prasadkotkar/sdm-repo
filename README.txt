Db first approach

1. Add Nuget Packages 
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.EntityFrameworkCore.Tools

2. Package Manager Console
 Scaffold-DbContext "Data Source=(localdb)\MsSqlLocalDb;Initial Catalog=Yash1;Integrated Security=true" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables Employee,Department

Note : read more at https://docs.microsoft.com/en-us/ef/core/managing-schemas/scaffolding?tabs=vs


3. Add EF service in Program.cs

        public static void Main(string[] args)
        {
            builder.Services.AddControllersWithViews();
            builder.Services.AddDbContext<Yash1Context>(options =>
                        options.UseSqlServer(builder.Configuration.GetConnectionString("Yash1Context")));

        }

4. Specify connection string in appsettings.json
"ConnectionStrings": {
    "Yash1Context": "Data Source=(localdb)\\MsSqlLocalDb;Initial Catalog=Yash1;Integrated Security=true;MultipleActiveResultSets=true"
  }

-----------------------------------------------------------
using Microsoft.AspNetCore.Mvc.ModelBinding;
using System.ComponentModel.DataAnnotations;

namespace DataAnnotations.Models
{
    public class Class1
    {
        [Key]
        [Display(Name = "Employee Number")]
        public int EmpNo { get; set; }

        [DataType(DataType.Text)]
        [Required(ErrorMessage = "Please enter name")]
        [StringLength(10, ErrorMessage = "The {0} value cannot exceed {1} characters. ")]
        public string Name { get; set; }
        
       [Range(1000, 500000, ErrorMessage = "Please enter values between 1000-500000")]
        [Display(Name = "Basic Salary")]
        public decimal Basic { get; set; }
        private decimal basic2;        
        public decimal Basic2 
        {
            get => basic2;
            set 
            {
                if (value < 100000)
                    basic2 = value;
                else
                {

                    throw new Exception("Invalid Basic");

                }
            } 
        }

        [MaxLength(6), MinLength(4)]
        public string DeptNo { get; set; }

        [AllowedValues("A", "B", "C")]
        [DeniedValues("D", "E")]
        public string AllowedValuesExample { get; set; }

        [ScaffoldColumn(false)]
        public string? Dummy { get; set; }

        [EmailAddress]
        public string EmailId { get; set; }

        [Required(ErrorMessage = "Please enter password")]
        [DataType(DataType.Password)]
        public string Password { get; set; }

        [Required(ErrorMessage = "Please enter confirm password")]
        [Compare("Password", ErrorMessage = "Password and confirm password should be the same")]
        [DataType(DataType.Password)]
        public string ConfirmPassword { get; set; }

        // Allow up to 40 uppercase and lowercase 
        // characters. Use custom error.
        [RegularExpression(@"^[a-zA-Z''-'\s]{1,40}$",
             ErrorMessage = "Characters are not allowed.")]
        public string FirstName { get; set; }

        // Allow up to 40 uppercase and lowercase 
        // characters. Use standard error.
        [RegularExpression(@"^[a-zA-Z''-'\s]{1,40}$")]
        public string LastName { get; set; }

        
    }
}

-----------------------------------------------------------
ModelMetadataType(typeof(EmployeeMetadata))]
    public class Employee
    {
        public int EmpNo { get; set; }
        public string Name { get; set; }

    }

public class EmployeeMetadata
    {
        [Display(Name = "Employee Number")]
        public int EmpNo { get; set; }

        [DataType(DataType.Text)]
        [Required(ErrorMessage = "Please enter name")]
        [StringLength(10, ErrorMessage = "The {0} value cannot exceed {1} characters. ")]
        public string Name { get; set; }

    }
-----------------------------------------------------------

Procedures:
CREATE PROCEDURE sp_RegisterUser
    @LoginName VARCHAR(50),
    @FullName VARCHAR(100),
    @Password VARCHAR(256),
    @Gender VARCHAR(10),
    @EmailId VARCHAR(100),
    @CityId INT,
    @PhoneNumber VARCHAR(15)
AS
BEGIN
    INSERT INTO Users (LoginName, FullName, Password, Gender, EmailId, CityId, PhoneNumber)
    VALUES (@LoginName, @FullName, @Password, @Gender, @EmailId, @CityId, @PhoneNumber)
END

CREATE PROCEDURE sp_ValidateUser
    @LoginName VARCHAR(50),
    @Password VARCHAR(256)
AS
BEGIN
    SELECT UserId, LoginName, FullName, Gender, EmailId, CityId, PhoneNumber
    FROM Users
    WHERE LoginName = @LoginName AND Password = @Password
END

CREATE PROCEDURE sp_GetAllUsers
    @CityFilter VARCHAR(50) = NULL
AS
BEGIN
    SELECT u.FullName, u.Gender, u.EmailId, u.PhoneNumber, c.CityName
    FROM Users u
    INNER JOIN Cities c ON u.CityId = c.CityId
    WHERE (@CityFilter IS NULL OR c.CityName LIKE '%' + @CityFilter + '%')
    ORDER BY u.FullName
END

CREATE PROCEDURE sp_UpdateUser
    @UserId INT,
    @FullName VARCHAR(100),
    @Gender VARCHAR(10),
    @EmailId VARCHAR(100),
    @CityId INT,
    @PhoneNumber VARCHAR(15)
AS
BEGIN
    UPDATE Users
    SET FullName = @FullName,
        Gender = @Gender,
        EmailId = @EmailId,
        CityId = @CityId,
        PhoneNumber = @PhoneNumber
    WHERE UserId = @UserId
END

--------------------------------------------------------
 public static void Update(Employee obj) //parameters
        {
            SqlConnection cn = new SqlConnection();
            cn.ConnectionString = "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=WEBAPI1;Integrated Security=True;";

            try
            {
                cn.Open();
                //SqlCommand cmd = cn.CreateCommand();
                SqlCommand cmd = new SqlCommand();
                cmd.Connection = cn;
                cmd.CommandType = CommandType.Text;
                cmd.CommandText =
                "update Employee set Name=@Name,Basic=@Basic,DeptNo=@DeptNo where EmpNo=@EmpNo";

                cmd.Parameters.AddWithValue("@EmpNo", obj.EmpNo);
                cmd.Parameters.AddWithValue("@Name", obj.Name);
                cmd.Parameters.AddWithValue("@Basic", obj.Basic);
                cmd.Parameters.AddWithValue("@DeptNo", obj.DeptNo);

                cmd.ExecuteNonQuery();
            }
            catch (Exception)
            {
                throw;
            }
            finally
            {
                cn.Close();
            }
        }

public static Employee GetSingleEmployee(int EmpNo)
        {
            Employee emp = null;

            SqlConnection cn = new SqlConnection();
            cn.ConnectionString = "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=WEBAPI1;Integrated Security=True;";
            try
            {
                cn.Open();
                SqlCommand cmd = new SqlCommand();
                cmd.Connection = cn;
                cmd.CommandType = CommandType.Text;
                cmd.CommandText = "select * from Employee where EmpNo=@EmpNo";
                cmd.Parameters.AddWithValue("@EmpNo", EmpNo);

                SqlDataReader dr = cmd.ExecuteReader();
                if (dr.Read())
                {

                    emp = new Employee();
                    emp.EmpNo = dr.GetInt32("EmpNo");
                    emp.Name = dr.GetString("Name");
                    emp.Basic = dr.GetDecimal("Basic");
                    emp.DeptNo = dr.GetInt32("DeptNo");
                }
                dr.Close();
            }
            catch (Exception)
            {
                throw;
            }
            finally
            {
                cn.Close();
            }

            return emp;
        }