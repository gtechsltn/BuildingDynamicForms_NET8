# Mastering Dynamic Forms in .NET 8: Unleash the Power of Flexibility and Customization

Mastering Dynamic Forms in .NET 8: Unleash the Power of Flexibility and Customization

https://medium.com/@michaelmaurice410/mastering-dynamic-forms-in-net-8-unleash-the-power-of-flexibility-and-customization-0b057c87a3a4

### Web API
```
dotnet new webapi -n DynamicFormsApp
cd DynamicFormsApp
```

### Entity Framework Core
```
dotnet add package Microsoft.EntityFrameworkCore 
dotnet add package Microsoft.EntityFrameworkCore.SqlServer 
dotnet add package Microsoft.EntityFrameworkCore.Tools
```

### Models
```
public class Form
{
    public int FormId { get; set; }
    public string FormName { get; set; }
    public string FormDescription { get; set; }
    public ICollection<FormField> Fields { get; set; }
}

public class FormField
{
    public int FieldId { get; set; }
    public int FormId { get; set; }
    public string FieldName { get; set; }
    public string FieldType { get; set; }
    public bool IsRequired { get; set; }
    public string Options { get; set; }
}

public class FormResponse
{
    public int ResponseId { get; set; }
    public int FormId { get; set; }
    public ICollection<FormFieldResponse> FieldResponses { get; set; }
}

public class FormFieldResponse
{
    public int FieldResponseId { get; set; }
    public int ResponseId { get; set; }
    public int FieldId { get; set; }
    public string ResponseValue { get; set; }
}
```

### DbContext
```
public class FormsContext : DbContext
{
    public DbSet<Form> Forms { get; set; }
    public DbSet<FormField> FormFields { get; set; }
    public DbSet<FormResponse> FormResponses { get; set; }
    public DbSet<FormFieldResponse> FormFieldResponses { get; set; }

    public FormsContext(DbContextOptions<FormsContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Form>()
            .HasMany(f => f.Fields)
            .WithOne()
            .HasForeignKey(ff => ff.FormId);

        modelBuilder.Entity<FormResponse>()
            .HasMany(fr => fr.FieldResponses)
            .WithOne()
            .HasForeignKey(fr => fr.ResponseId);
    }
}
```

### Database Connection String in appsettings.json
```
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=DynamicFormsDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

### Program.cs
```
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<FormsContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

### Migrations
```
dotnet ef migrations add InitialCreate 
dotnet ef database update
```

### APIs
```
[ApiController]
[Route("api/[controller]")]
public class FormsController : ControllerBase
{
    private readonly FormsContext _context;

    public FormsController(FormsContext context)
    {
        _context = context;
    }

    [HttpPost]
    public async Task<IActionResult> CreateForm([FromBody] Form form)
    {
        _context.Forms.Add(form);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetForm), new { id = form.FormId }, form);
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<Form>> GetForm(int id)
    {
        var form = await _context.Forms.Include(f => f.Fields).FirstOrDefaultAsync(f => f.FormId == id);
        if (form == null)
        {
            return NotFound();
        }
        return form;
    }
}
```

### Final Thoughts

Do you know how empowering it is to create a system that adapts to its users? Dynamic forms not only enhance user experience but also provide a scalable solution for data collection needs. If you’re interested in crafting applications that are both innovative and user-centric, you should delve deeper into .NET 8 and its powerful features for building dynamic forms.

By sharing this article, you’re not just spreading knowledge — you’re inviting others to explore a world where forms aren’t just static fields but dynamic tools of interaction. Imagine the possibilities and start building your dynamic forms system today!
