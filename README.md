



public partial class MyPage : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!IsPostBack)
        {
            ListarGarantias();
        }
    }

    private void ListarGarantias()
    {
        List<cPruebaClase> vlista = new List<cPruebaClase>()
        {
            new cPruebaClase { Tipo = "Tipo 1", Propietarios = "106000023", Titular = "John Doe", Asignar = true, Garantia = "Garantia1", NroPartidaRegistral = "123456", OfiRegistral = "Oficina1", Moneda = "USD", Montolinea = "123", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 1", Propietarios = "106000026", Titular = "Jane Smith", Asignar = false, Garantia = "Garantia2", NroPartidaRegistral = "654321", OfiRegistral = "Oficina2", Moneda = "EUR", Montolinea = "456", Disponible = "No", Gravado = "Si" },
            new cPruebaClase { Tipo = "Tipo 2", Propietarios = "106000030", Titular = "Alice Johnson", Asignar = true, Garantia = "Garantia3", NroPartidaRegistral = "987654", OfiRegistral = "Oficina3", Moneda = "USD", Montolinea = "789", Disponible = "No", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 2", Propietarios = "106000031", Titular = "Alice Johnson", Asignar = true, Garantia = "Garantia4", NroPartidaRegistral = "456789", OfiRegistral = "Oficina4", Moneda = "EUR", Montolinea = "012", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000040", Titular = "Alberto j", Asignar = false, Garantia = "Garantia5", NroPartidaRegistral = "159753", OfiRegistral = "Oficina5", Moneda = "USD", Montolinea = "345", Disponible = "No", Gravado = "Si" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000041", Titular = "Alberto j", Asignar = true, Garantia = "Garantia6", NroPartidaRegistral = "357159", OfiRegistral = "Oficina6", Moneda = "EUR", Montolinea = "678", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000042", Titular = "Alberto j", Asignar = false, Garantia = "Garantia7", NroPartidaRegistral = "951357", OfiRegistral = "Oficina7", Moneda = "USD", Montolinea = "901", Disponible = "Si", Gravado = "Si" }
        };

        Repeater1.DataSource = vlista.GroupBy(x => x.Tipo).ToList();
        Repeater1.DataBind();
    }

    protected void Repeater1_ItemDataBound(object sender, RepeaterItemEventArgs e)
    {
        if (e.Item.ItemType == ListItemType.Item || e.Item.ItemType == ListItemType.AlternatingItem)
        {
            var dropdown = (DropDownList)e.Item.FindControl("PropietariosDropdown");
            var item = (IGrouping<string, cPruebaClase>)e.Item.DataItem;
            dropdown.Items.Add(new ListItem("0", "Propietario 0"));

            foreach (var cPruebaClase in item)
            {
                dropdown.Items.Add(new ListItem(cPruebaClase.Propietarios, cPruebaClase.Propietarios));
            }
        }
    }
}

public class cPruebaClase
{
    public string Tipo { get; set; }
    public string Titular { get; set; }
    public bool Asignar { get; set; }
    public string Garantia { get; set; }
    public string NroPartidaRegistral { get; set; }
    public string PersCod { get; set; }
    public string OfiRegistral { get; set; }
    public string Moneda { get; set; }
    public string Montolinea { get; set; }
    public string Disponible { get; set; }
    public string Gravado { get; set; }
    public string Propietarios { get; set; }
}

<asp:Repeater ID="Repeater1" runat="server" OnItemDataBound="Repeater1_ItemDataBound">
    <ItemTemplate>
        <table border="1">
            <tr>
                <th colspan="10">Titular (<%# ((IGrouping<string, cPruebaClase>)Container.DataItem).Key %>)</th>
            </tr>
            <tr>
                <td>Asignar</td>
                <td>Garantia</td>
                <td>NroPartidaRegistral</td>
                <td colspan="2">OfiRegistral</td>
                <td>Moneda</td>
                <td>Montolinea</td>
                <td>Disponible</td>
                <td>Gravado</td>
                <td>
                    <asp:DropDownList ID="PropietariosDropdown" runat="server"></asp:DropDownList>
                </td>
            </tr>
            <asp:Repeater ID="SubItemRepeater" runat="server">
                <ItemTemplate>
                    <tr>
                        <td><%# Eval("Asignar") %></td>
                        <td><%# Eval("Garantia") %></td>
                        <td><%# Eval("NroPartidaRegistral") %></td>
                        <td colspan="2"><%# Eval("OfiRegistral") %></td>
                        <td><%# Eval("Moneda") %></td>
                        <td><%# Eval("Montolinea") %></td>
                        <td><%# Eval("Disponible") %></td>
                        <td><%# Eval("Gravado") %></td>
                        <td>
                            <asp:DropDownList ID="PropietariosDropdown" runat="server"></asp:DropDownList>
                        </td>
                    </tr>
                </ItemTemplate>
            </asp:Repeater>
        </table>
    </ItemTemplate>
</asp:Repeater>



using System;
using System.Collections.Generic;
using System.Web.UI.WebControls;

public partial class MyPage : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!IsPostBack)
        {
            ListarGarantias();
            DropDownListPropietarios.DataSource = ListarValores();
            DropDownListPropietarios.DataBind();
            DropDownListPropietarios.Items.Insert(0, new ListItem("Propietarios", "0"));
        }
    }

    private void ListarGarantias()
    {
        List<cPruebaClase> vlista = new List<cPruebaClase>()
        {
            new cPruebaClase { Tipo = "Tipo 1", Propietarios = "106000023", Titular = "John Doe", Asignar = true, Garantia = "Garantia1", NroPartidaRegistral = "123456", OfiRegistral = "Oficina1", Moneda = "USD", Montolinea = "123", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 1", Propietarios = "106000026", Titular = "Jane Smith", Asignar = false, Garantia = "Garantia2", NroPartidaRegistral = "654321", OfiRegistral = "Oficina2", Moneda = "EUR", Montolinea = "456", Disponible = "No", Gravado = "Si" },
            new cPruebaClase { Tipo = "Tipo 2", Propietarios = "106000030", Titular = "Alice Johnson", Asignar = true, Garantia = "Garantia3", NroPartidaRegistral = "987654", OfiRegistral = "Oficina3", Moneda = "USD", Montolinea = "789", Disponible = "No", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 2", Propietarios = "106000031", Titular = "Alice Johnson", Asignar = true, Garantia = "Garantia4", NroPartidaRegistral = "456789", OfiRegistral = "Oficina4", Moneda = "EUR", Montolinea = "012", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000040", Titular = "Alberto j", Asignar = false, Garantia = "Garantia5", NroPartidaRegistral = "159753", OfiRegistral = "Oficina5", Moneda = "USD", Montolinea = "345", Disponible = "No", Gravado = "Si" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000041", Titular = "Alberto j", Asignar = true, Garantia = "Garantia6", NroPartidaRegistral = "357159", OfiRegistral = "Oficina6", Moneda = "EUR", Montolinea = "678", Disponible = "Si", Gravado = "No" },
            new cPruebaClase { Tipo = "Tipo 3", Propietarios = "106000042", Titular = "Alberto j", Asignar = false, Garantia = "Garantia7", NroPartidaRegistral = "951357", OfiRegistral = "Oficina7", Moneda = "USD", Montolinea = "901", Disponible = "Si", Gravado = "Si" }
        };

        // Agrupar por tipo
        var gruposPorTipo = vlista.GroupBy(item => item.Tipo);

        foreach (var grupo in gruposPorTipo)
        {
            // Crear tabla para el grupo actual
            var tabla = new System.Web.UI.WebControls.Table();
            tabla.CssClass = "tabla-garantias";

            // Agregar encabezado
            var encabezado = new TableHeaderRow();
            encabezado.Cells.Add(new TableHeaderCell { Text = "Tipo" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Titular" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Asignar" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Garantia" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "NroPartidaRegistral" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "OfiRegistral" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Moneda" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Montolinea" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Disponible" });
            encabezado.Cells.Add(new TableHeaderCell { Text = "Gravado" });
            tabla.Rows.Add(encabezado);

            // Agregar filas para cada elemento del grupo
            foreach (var item in grupo)
            {
                var fila = new TableRow();
                fila.Cells.Add(new TableCell { Text = item.Tipo });
                fila.Cells.Add(new TableCell { Text = item.Titular });
                fila.Cells.Add(new TableCell { Text = item.Asignar.ToString() });
                fila.Cells.Add(new TableCell { Text = item.Garantia });
                fila.Cells.Add(new TableCell { Text = item.NroPartidaRegistral });
                fila.Cells.Add(new TableCell { Text = item.OfiRegistral });
                fila.Cells.Add(new TableCell { Text = item.Moneda });
                fila.Cells.Add(new TableCell { Text = item.Montolinea });
                fila.Cells.Add(new TableCell { Text = item.Disponible });
                fila.Cells.Add(new TableCell { Text = item.Gravado });
                tabla.Rows.Add(fila);
            }

            // Agregar la tabla al formulario
            form1.Controls.Add(tabla);
        }
    }

    protected void DropDownListPropietarios_SelectedIndexChanged(object sender, EventArgs e)
    {
        string propietarioSeleccionado = DropDownListPropietarios.SelectedValue;
        bool asignar = true; // Valor predeterminado
        // Lógica para recuperar datos relacionados con el propietario seleccionado
        // Puedes realizar acciones adicionales aquí, como mostrar información específica o realizar otras operaciones.
    }

    public List<ListItem> ListarValores(bool SuperaMonto = false)
    {
        List<ListItem> items = new List<ListItem>();
        if (!SuperaMonto)
        {
            items.Add(new ListItem("Analista", "106000023"));
        }
        items.Add(new ListItem("Usuario 1", "106000026"));
        items.Add(new ListItem("Usuario 2", "106000029"));
        return items;
    }
}

public class cPruebaClase
{
    public string Tipo { get; set; }
    public string Titular { get; set; }
    public bool Asignar { get; set; }
    public string Garantia { get; set; }
    public string NroPartidaRegistral { get; set; }
    public string PersCod { get; set; }
    public string OfiRegistral { get; set; }
    public string Moneda { get; set; }
    public string Montolinea { get; set; }
    public string Disponible { get; set; }
    public string Gravado { get; set; }
    public string Propietarios { get; set; }
}





