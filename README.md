Microsoft.EntityFrameworkCore.Tools

MySql.EntityFrameworkCore

Scaffold-DbContext "server=localhost;database=db neve;user=root;" Mysql.EntityFrameworkCore -Project project neve

CommunityToolkit.Mvvm

********************************************
Iskola
********************************************
Student.cs

using System;
using System.Collections.Generic;

namespace iskola_gui;

public partial class Student
{
    public ulong Id { get; set; }

    public string Vezeteknev { get; set; } = null!;

    public string Keresztnev { get; set; } = null!;

    public string Email { get; set; } = null!;

    public DateTime? SzuletesiDatum { get; set; }

    public ulong OsztalyId { get; set; }

    public virtual SchoolClass Osztaly { get; set; } = null!;

    public Student() { }
    
    public Student(ulong id, string vezeteknev, string keresztnev, string email, DateTime? szuletesiDatum, ulong osztalyId, SchoolClass osztaly)
    {
        Id = id;
        Vezeteknev = vezeteknev;
        Keresztnev = keresztnev;
        Email = email;
        SzuletesiDatum = szuletesiDatum;
        OsztalyId = osztalyId;
        Osztaly = osztaly;
    }

    public override string? ToString()
    {
        return Vezeteknev+" "+Keresztnev;
    }
}
******************************
SchoolClass.cs

using System;
using System.Collections.Generic;

namespace iskola_gui;

public partial class SchoolClass
{
    public ulong Id { get; set; }

    public string Nev { get; set; } = null!;

    public virtual ICollection<Student> Students { get; set; } = new List<Student>();

    public SchoolClass() { }

    public SchoolClass(ulong id, string nev, ICollection<Student> students)
    {
        Id = id;
        Nev = nev;
        Students = students;
    }
}
************************************
ViewMosel.cs

////Egyirányú adatkötés

namespace iskola_gui
{
    public class ViewModel
    {
        public ObservableCollection<Student> Students { get; set; }
        public ObservableCollection<SchoolClass> SchoolClasses { get; set; }

        public Student Kivalasztott { get; set; }
               
        public ViewModel()
        {
            using WpfIskolaContext adatbazis = new WpfIskolaContext();
            Students = new (adatbazis.Students); 
            SchoolClasses = new (adatbazis.SchoolClasses); 
        }
    }
}

////Kétirányú adatkötés

namespace iskola_gui
{
    internal partial class ViewModel:ObservableObject
    {
        [ObservableProperty]
        ObservableCollection<Student> students;
        
        [ObservableProperty]
        Student kivalasztott;

        [ObservableProperty]
        int darab;

        private readonly WpfIskolaContext db = new WpfIskolaContext();

        public ViewModel()
        {
            students = new ObservableCollection<Student>(db.Students);
        }

        // A CommunityToolkit ezt a metódust automatikusan hívja,
        // amikor a Kivalasztott megváltozik
        partial void OnKivalasztottChanged(Student value)
        {
            if (value != null)
            {
                // Csak a kiválasztott diák Osztályát tölti be
                db.Entry(value).Reference(x => x.Osztaly).Load();
            }
        }
        [RelayCommand]
        public void Torles()
        {
            if (Kivalasztott == null)
                return;

            db.Students.Remove(Kivalasztott);
            db.SaveChanges();

            Students.Remove(Kivalasztott);
        }

        [RelayCommand]
        public void Szamolas()
        {
            if (Kivalasztott == null)
                return;

            Darab = db.Students.Count(x => x.OsztalyId == Kivalasztott.OsztalyId);
        }

        [RelayCommand]
        public void ExportStudentsJson()
        {
            var options = new JsonSerializerOptions
            {
                WriteIndented = true,
                Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping,
                ReferenceHandler = System.Text.Json.Serialization.ReferenceHandler.IgnoreCycles
            };

            var lista = Students.ToList();

            // csak pl. a 9A osztályba járók
            var csak9A = Students
                .Where(x => x.Osztaly?.Nev == "9A")
                .ToList();

            string json = JsonSerializer.Serialize(lista, options);

            // fájl mentése
            File.WriteAllText("students.json", json, Encoding.UTF8);
        }
    }
}

*************************************
MainWindiw.xaml

<Window x:Class="iskola_gui.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:iskola_gui"
        mc:Ignorable="d"
        Title="Iskola" Height="450" Width="600">

    <Window.DataContext>
    <local:ViewModel/>
    </Window.DataContext>

    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="1*"/>
            <ColumnDefinition Width="2*"/>
        </Grid.ColumnDefinitions>

        <ListBox
            Grid.Column="0"
            ItemsSource="{Binding Students}"
            SelectedItem="{Binding Kivalasztott}"/>

        <StackPanel Grid.Column="1" Margin="10 0">
            <Label Content="Vezetéknév"/>
            <TextBox Text="{Binding Kivalasztott.Vezeteknev}"/>
            <Label Content="Keresztnév"/>
            <TextBox Text="{Binding Kivalasztott.Keresztnev}"/>
            <Label Content="Email cím"/>
            <TextBox Text="{Binding Kivalasztott.Email}"/>
            <Label Content="Születési dátum"/>
            <TextBox Text="{Binding Kivalasztott.SzuletesiDatum}"/>
            <Label Content="Osztály"/>
            <TextBox Text="{Binding Kivalasztott.Osztaly.Nev}"/>
            <Button 
                Content="Törlés" 
                Margin="0 15"
                Command="{Binding TorlesCommand}"/>
            <TextBox Text="{Binding Darab}"/>
            <Button 
                Content="Hányan járnak az aktuális osztályba?" 
                Margin="0 5"
                Command="{Binding SzamolasCommand }"/>
            <Button 
                Content="Mentés" 
                Margin="0 15"
                Command="{Binding ExportStudentsJsonCommand }"/>

        </StackPanel>

    </Grid>
</Window>




**************************************
realestate.cs
**************************************

namespace GUI;

public partial class Realestate
{
    public long Id { get; set; }

    public long CategoryId { get; set; }

    public long SellerId { get; set; }

    public string? Description { get; set; }

    public DateTime CreateAt { get; set; }

    public bool Freeofcharge { get; set; }

    public string ImageUrl { get; set; } = null!;

    public int? Area { get; set; }

    public int? Rooms { get; set; }

    public int? Floors { get; set; }

    public string? Latlong { get; set; }

    public virtual Category Category { get; set; } = null!;

    public virtual Seller Seller { get; set; } = null!;

    public override string ToString()
    {
        return $"URL: { ImageUrl}";
    }
}
****************************************************
ViewModel.cs
****************************************************
namespace GUI
{
    internal partial class ViewModel: ObservableObject
    {
        [ObservableProperty]
        ObservableCollection<Realestate> realestates;

        [ObservableProperty]
        Realestate kivalasztott;

        [ObservableProperty]
        int darab;

        public ViewModel() 
        {
            using IngatlanokContext adatbazis = new();
        
            realestates = new(adatbazis.Realestates);
        }

        [RelayCommand]
        public void Megszamol()
        {
            using IngatlanokContext db = new();

            Darab = db.Categories.Where(x => x.Id == Kivalasztott.CategoryId).Count();
        }

        [RelayCommand]
        public void Torles()
        {
            using IngatlanokContext db = new();

            db.Realestates.Remove(Kivalasztott);
        }
    }
}
*********************************************
MainWindows.xaml
*********************************************
<Window x:Class="GUI.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:GUI"
        mc:Ignorable="d"
        Title="MainWindow" 
        Height="450" 
        Width="400">

    <Window.DataContext>
        <local:ViewModel/>
    </Window.DataContext>
    
    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="1*"/>
            <ColumnDefinition Width="2*"/>
        </Grid.ColumnDefinitions>

        <ListBox
            Grid.Column="0"
            ItemsSource="{Binding Realestates}"
            SelectedItem="{Binding Kivalasztott}"
            />

        <StackPanel Grid.Column="1" Margin="10">
            <Label Content="Leírás"/>
            <TextBox
                Text="{Binding Kivalasztott.ImageUrl}"/>
            
            <Label Content="Kategória"/>
            <TextBox
                Text="{Binding Kivalasztott.Latlong}"/>

            <Button Content="Betöltés" Margin="0,10,0,10" IsEnabled="False"/>
            <Label/>
            
        </StackPanel>

    </Grid>
</Window>


