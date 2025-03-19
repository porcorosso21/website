# UserControl事件的建立與外部綁定

在UserControl中建立一個UcClick事件，讓UserControl中的Control在Click時，能夠觸發UcClick這個事件；並讓外部的Form能夠綁定UcClick事件並執行後續的動作。

```csharp
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApp1
{
    public partial class Form1: Form
    {
        public Form1()
        {
            InitializeComponent();

            //建立一個UserControl
            Uc uc = new Uc();
            uc.Location = new Point(10, 10);
            uc.Size = new Size(100, 100);
            //綁定UcClick事件觸發時，顯示訊息Uc的Tag
            uc.UcClick += (s, e) =>
            {
                MessageBox.Show("Uc Tage is " + ((Uc)s).Tag.ToString());
            };
            this.Controls.Add(uc);

        }
    }

    /// <summary>
    /// UserControl
    /// </summary>
    public class Uc : UserControl
    {
        /// <summary>
        /// EventHandler
        /// </summary>
        public EventHandler UcClick;

        /// <summary>
        /// 建構
        /// </summary>
        public Uc()
        {
            this.Tag = "UcTag";

            //建立一個Label
            Label lb = new Label();
            lb.Text = "Click Me";
            lb.TextAlign = ContentAlignment.MiddleCenter;
            lb.BackColor = Color.White;
            //綁定Click時觸發UserControl的UcClick事件
            lb.Click += (s, e) =>
            {
                UcClick?.Invoke(this, e); //這邊使用this是把Uc傳遞出去
            };
            lb.AutoSize = false;
            lb.Dock = DockStyle.Fill;
            this.Controls.Add(lb);
        }
    }
}

```
