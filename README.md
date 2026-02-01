Microsoft.EntityFrameworkCore.Tools
MySql.EntityFrameworkCore

Scaffold-DbContext "server=localhost;database=db neve;user=root;" Mysql.EntityFrameworkCore -Project project neve

CommunityToolkit.Mvvm

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
