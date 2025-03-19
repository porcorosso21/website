# SQLite

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Text;
using System.Data.SQLite;

namespace ConsoleApp1
{
    internal class Program
    {
        static void Main(string[] args)
        {
            //A~Z亂數產生
            int i = new Random().Next(25) + 65;
            String EmpName = "Mr." + ((char)i).ToString();

            SQLite sql = new SQLite("test.db");

            //CREATE TABLE
            sql.BeginTransaction();
            sql.CommandClear();
            sql.CommandAdd("CREATE TABLE IF NOT EXISTS Employee (id INTEGER PRIMARY KEY AUTOINCREMENT, EmpName TEXT)");
            sql.NonQuery();

            //INSERT
            sql.CommandClear();
            sql.CommandAdd("INSERT INTO Employee ( EmpName");
            sql.CommandAdd("        ) VALUES (");
            sql.CommandAdd("                  @EmpName);");
            sql.ParamAdd("EmpName", EmpName, DbType.String);
            sql.NonQuery();
            sql.Commit();

            //SELECT
            sql.CommandClear();
            sql.CommandAdd(" SELECT *  FROM {0}", "Employee");
            DataTable dt = sql.Query();
            Console.WriteLine("First Employee EmpName : " + dt.Rows[0]["EmpName"].ToString());
            Console.WriteLine("Last Employee EmpName : " + dt.Rows[dt.Rows.Count - 1]["EmpName"].ToString());
            Console.WriteLine("Employee Count : " + dt.Rows.Count.ToString());
        }

        /// <summary>
        /// SQLite
        /// 使用NuGet安裝System.Data.SQLite會在專案中產生System.Data.SQLite.dll
        /// 並建立x86、x64y資料夾，將SQLite.Interop.dll放入
        /// SQLite需要區分x86、x64，否則會出現錯誤
        /// 版本1.0.111.0是最後支援資料庫加密的版本，若需資料庫加密，請使用此版本
        /// </summary>
        public class SQLite
        {
            /// <summary>SQLiteConnection</summary>
            private SQLiteConnection con;

            /// <summary>SQLiteTransaction</summary>
            private SQLiteTransaction trans;

            /// <summary>CommandTimeout</summary>
            private int commandtimeout { get { return 60; } }

            /// <summary>SQLite Commands</summary>
            private StringBuilder commands = new StringBuilder();

            /// <summary>Parameters Class</summary>
            private class Param
            {
                public String Name;
                public object Obj;
                public DbType DbType;
            }

            /// <summary>Parameters清單</summary>
            private List<Param> paramlist = new List<Param>();

            /// <summary>
            /// 建構
            /// </summary>
            /// <param name="db"></param>
            public SQLite(String db)
            {
                SQLiteConnectionStringBuilder scsb = new SQLiteConnectionStringBuilder();
                scsb.DataSource = db;
                scsb.Version = 3; //版本
                //scsb.Password = password; //如果資料庫有加密在這邊設定密碼
                con = new SQLiteConnection(scsb.ConnectionString);
            }

            /// <summary>
            /// 清除指令與變數
            /// </summary>
            public void CommandClear()
            {
                commands.Clear();
                paramlist.Clear();
            }

            /// <summary>
            /// 添加指令，參數前需加@
            /// </summary>
            /// <param name="command"></param>
            public void CommandAdd(String command)
            {
                commands.AppendLine(command);
            }

            /// <summary>
            ///  添加指令，參數前需加@
            /// </summary>
            /// <param name="format"></param>
            /// <param name="args"></param>
            public void CommandAdd(String format, params String[] args)
            {
                commands.AppendLine(String.Format(format, args));
            }

            /// <summary>
            /// 添加參數
            /// </summary>
            /// <param name="name">參數名稱，前面不用加@</param>
            /// <param name="obj">參數值</param>
            /// <param name="dbtype">資料型態</param>
            public void ParamAdd(String name, Object obj, System.Data.DbType dbtype)
            {
                Param p = new Param();
                p.Name = name;
                p.Obj = obj;
                p.DbType = dbtype;

                paramlist.Add(p);
            }

            /// <summary>
            /// SELECT
            /// </summary>
            /// <returns></returns>
            /// <exception cref="Exception"></exception>
            public DataTable Query()
            {
                try
                {
                    SQLiteCommand cmd = new SQLiteCommand(commands.ToString(), con);
                    cmd.CommandTimeout = this.commandtimeout;
                    cmd.CommandType = CommandType.Text;

                    setParams(ref cmd);

                    SQLiteDataAdapter da = new SQLiteDataAdapter(cmd);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    cmd.Dispose();
                    return dt;
                }
                catch (Exception ex)
                {
                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[SQLite]");
                    exceptionMessage.AppendLine(commands.ToString());
                    exceptionMessage.AppendLine(String.Empty);
                    exceptionMessage.AppendLine("---------------");
                    exceptionMessage.AppendLine(String.Empty);
                    exceptionMessage.AppendLine(ex.Message);
                    throw new Exception(exceptionMessage.ToString(), ex);
                }
            }

            /// <summary>
            /// BeginTransaction
            /// </summary>
            /// <exception cref="Exception"></exception>
            public void BeginTransaction()
            {
                try
                {
                    if (con.State != ConnectionState.Open)
                    {
                        con.Open();
                    }
                    this.trans = con.BeginTransaction();
                }
                catch (Exception ex)
                {
                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[SQLite BeginTransaction]");
                    exceptionMessage.AppendLine(ex.Message);
                    throw new Exception(exceptionMessage.ToString(), ex);
                }
            }

            /// <summary>
            /// Commit
            /// </summary>
            /// <exception cref="Exception"></exception>
            public void Commit()
            {
                try
                {
                    trans.Commit();
                    con.Clone();
                }
                catch (Exception ex)
                {
                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[SQLite Commit]");
                    exceptionMessage.AppendLine(ex.Message);
                    throw new Exception(exceptionMessage.ToString(), ex);
                }
            }

            /// <summary>
            /// Rollback
            /// </summary>
            /// <exception cref="Exception"></exception>
            public void Rollback()
            {
                try
                {
                    trans.Rollback();
                    con.Clone();
                }
                catch (Exception ex)
                {
                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[SQLite Rollback]");
                    exceptionMessage.AppendLine(ex.Message);
                    throw new Exception(exceptionMessage.ToString(), ex);
                }
            }

            /// <summary>
            /// UPDATE INSERT DELETE
            /// </summary>
            /// <exception cref="Exception"></exception>
            public void NonQuery()
            {
                try
                {
                    SQLiteCommand cmd = new SQLiteCommand(commands.ToString(), con);
                    cmd.CommandTimeout = this.commandtimeout;
                    cmd.CommandType = CommandType.Text;
                    cmd.Transaction = this.trans;

                    setParams(ref cmd);

                    cmd.ExecuteNonQuery();
                    cmd.Dispose();
                }
                catch (Exception ex)
                {
                    Rollback();

                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[SQLite]");
                    exceptionMessage.AppendLine(commands.ToString());
                    exceptionMessage.AppendLine(String.Empty);
                    exceptionMessage.AppendLine("---------------");
                    exceptionMessage.AppendLine(String.Empty);
                    exceptionMessage.AppendLine(ex.Message);
                    throw new Exception(exceptionMessage.ToString(), ex);
                }
            }

            /// <summary>
            /// 添加參數
            /// </summary>
            /// <param name="cmd"></param>
            private void setParams(ref SQLiteCommand cmd)
            {
                cmd.Parameters.Clear();

                foreach (Param p in paramlist)
                {
                    cmd.Parameters.Add(p.Name, p.DbType).Value = p.Obj;
                }
            }
        }
    }

```
