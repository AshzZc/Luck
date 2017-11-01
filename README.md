# Luck
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.IO;
using System.Data.SQLite;


namespace HappyLuck
{
    public partial class Form1 : Form
    {
        string CSV_FilePath = "";//用来记录当前打开文件的路径的

        /// <summary>
        /// 人员列表
        /// </summary>
        List<UserBaseInfo> userbaseInfoList = new List<UserBaseInfo>();

        bool bol_Time1Enb = false;
        bool bol_Time2Enb = false;

        public Form1()
        {
            InitializeComponent();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
            mFullScreen();
            this.menuStrip1.BackColor = Color.Transparent;
        }

        private void mFullScreen()
        {
            try
            {
                this.WindowState = FormWindowState.Maximized;
                this.FormBorderStyle = FormBorderStyle.Sizable;

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.ToString());
            }
        }

        private void Menu_MesCon_Click(object sender, EventArgs e)
        {
            OpenFileDialog openFileDialog1 = new System.Windows.Forms.OpenFileDialog();//一个打开文件的对话框  
            openFileDialog1.Filter = "csv文件(*.csv)|*.csv";//设置允许打开的扩展名  
            if (openFileDialog1.ShowDialog() == DialogResult.OK)//判断是否选择了文件    
            {
                CSV_FilePath = openFileDialog1.FileName;//记录用户选择的文件路径  
            }
            try
            {
                StreamReader sr = new StreamReader(CSV_FilePath, System.Text.Encoding.Default);
                userbaseInfoList.Clear();
                while (!sr.EndOfStream)
                {
                    string strTmp = sr.ReadLine();

                    string[] _tmp = strTmp.Split(',');

                    UserBaseInfo _u = new UserBaseInfo();
                    _u.id = _tmp[0];
                    _u.name = _tmp[1];
                    _u.even = _tmp[2];
                    _u.score1 = _tmp[3];
                    _u.score2 = _tmp[4];
                    _u.date = _tmp[5];

                    userbaseInfoList.Add(_u);
                }
                sr.Close();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.ToString());
                return;
            }
            MessageBox.Show("载入成功！");
            timer1.Start();
            timer2.Start();

        }

        private void timer1_Tick(object sender, EventArgs e)
        {
            if (bol_Time1Enb == true)
            {
                for (int i=1;i<=w_02_nud.Value;i++)
                {
                    Random r = new Random(Guid.NewGuid().GetHashCode());
                    int x = r.Next(0, userbaseInfoList.Count);
                    Label lb = (Label)Controls.Find("lblname"+i.ToString(), false)[0];
                    lb.Text = userbaseInfoList[x].name; ;
                    Label lb2 = (Label)Controls.Find("lbleven" + i.ToString(), false)[0];
                    lb2.Text = string.Format("动分：{0}\r\n原因：{1}", userbaseInfoList[x].score2, userbaseInfoList[x].even);
                }
            }
          
        }

        private void btnonoff_Click(object sender, EventArgs e)
        {
            if (userbaseInfoList.Count == 0)
            {
                MessageBox.Show("请载入人员！");
                return;
            }

            if(w_02_nud.Value<=0|| w_02_nud.Value>20)
            {
                MessageBox.Show("请设置人数（1——20）！");
                return;
            }

            btnonoff.Enabled = false;
            bol_Time2Enb = true;

            if (bol_Time1Enb == true)
            {
                bol_Time1Enb = false;
                btnonoff.Text = "开始";
                for (int i = 1; i <= w_02_nud.Value; i++)
                {
                    Random r = new Random(Guid.NewGuid().GetHashCode());
                    int x = r.Next(0, userbaseInfoList.Count);
                    Label lb = (Label)Controls.Find("lblname" + i.ToString(), false)[0];
                    lb.Text = userbaseInfoList[x].name; ;
                    Label lb2 = (Label)Controls.Find("lbleven" + i.ToString(), false)[0];
                    lb2.Text = string.Format("动分：{0}\r\n原因：{1}", userbaseInfoList[x].score2, userbaseInfoList[x].even);
                    saveData2(userbaseInfoList[x].name);
                    userbaseInfoList.RemoveAt(x);
                }
             
            }
            else
            {
                bol_Time1Enb = true;
                btnonoff.Text = "停止";
            }
        }


        private void timer2_Tick(object sender, EventArgs e)
        {
            if (bol_Time2Enb == true)
            {
                btnonoff.Enabled = true;
                bol_Time2Enb = false;
            }
        }

        private void Form1_KeyDown(object sender, KeyEventArgs e)
        {
            if ((e.KeyCode == Keys.Space || e.KeyCode == Keys.Enter) && btnonoff.Enabled == true)
            {
                btnonoff_Click(null, null);
            }
        }

        private void btnresult_Click(object sender, EventArgs e)
        {
            Form2 form2 = new Form2();
            form2.ShowDialog();
        }

        private void saveData2(string str_PeopleName)
        {
            sqlite sqlStr = new sqlite();

            SQLiteConnection DBConn = new SQLiteConnection(sqlStr.sqliteStr);

            int queryCount = 0;
            try
            {
                DBConn.Open();
                SQLiteCommand myComm = new SQLiteCommand(sqlStr.InsertRecord2(str_PeopleName), DBConn);
                queryCount = myComm.ExecuteNonQuery();

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.ToString());
            }
            finally
            {
                DBConn.Close();
            }
        }

        private void Bt_Send_Click(object sender, EventArgs e)
        {
            if (w_02_nud.Enabled == false)
            {
                w_02_nud.Enabled = true;
            }
            else
            {
                w_02_nud.Enabled = false;
            }
        }
    }
}
