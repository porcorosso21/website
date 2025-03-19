# 資料庫連線

## MySql/MariaDB

```csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Text;
using MySql.Data.MySqlClient;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //A~Z亂數產生
            int i = new Random().Next(25) + 65;
            String EmpName = "Mr." + ((char)i).ToString();

            //INSERT
            MySQL sql = new MySQL();
            sql.CommandClear();
            sql.BeginTransaction();
            sql.CommandAdd("INSERT INTO employee (EmpName, EmpBirthday) VALUES (@EmpName, @EmpBirthday);");
            sql.ParamAdd("EmpName", EmpName, MySQL.ParamType.String);
            sql.ParamAdd("EmpBirthday", DateTime.Now.AddDays(Convert.ToInt32(new Random().Next(9))).ToString("yyyy-MM-dd"), MySQL.ParamType.DateTime); //亂數產生日期
            sql.NonQuery();
            sql.Commit();

            Console.WriteLine("Last Inserted Id: " + sql.LastInsertedId);

            //SELECT
            sql.CommandClear();
            sql.CommandAdd(" SELECT *  FROM employee");
            DataTable dt = sql.Query();

            Console.WriteLine("First Employee EmpName : " + dt.Rows[0]["EmpName"].ToString());
            Console.WriteLine("Last Employee EmpName : " + dt.Rows[dt.Rows.Count - 1]["EmpName"].ToString());
        }

        /// <summary>
        /// MySQL
        /// </summary>
        public class MySQL
        {
            /// <summary>
            /// Parameters資料型態
            /// 對應MySqlDbType，可依需求增加
            /// </summary>
            public enum ParamType
            {
                DateTime,
                Int32,
                String,
            }

            /// <summary>
            /// Last Inserted Id
            /// </summary>
            public long LastInsertedId
            {
                get { return lastInsertedId; }
            }
            private long lastInsertedId;

            /// <summary>Server</summary>
            private String server { get { return "localhost"; } }

            /// <summary>DataBase</summary>
            private String database { get { return "test"; } }

            /// <summary>User ID</summary>
            private String userID { get { return "root"; } }

            /// <summary>Password</summary>
            private String password { get { return "root"; } }

            /// <summary>MySqlConnection</summary>
            private MySqlConnection con;

            /// <summary>MySqlTransaction</summary>
            private MySqlTransaction trans;

            /// <summary>CommandTimeout</summary>
            private int cmdTimeoutSecond { get { return 60; } }

            /// <summary>MySQL Commands</summary>
            private StringBuilder commands = new StringBuilder();

            /// <summary>Parameters Class</summary>
            private class Param
            {
                public String Name;
                public object Obj;
                public ParamType ParamType;
            }

            /// <summary>Parameters清單</summary>
            private List<Param> paramlist = new List<Param>();

            /// <summary>
            /// 建構
            /// </summary>
            public MySQL()
            {
                MySqlConnectionStringBuilder mscs = new MySqlConnectionStringBuilder();
                mscs.Server = this.server;
                mscs.Database = this.database;
                mscs.UserID = this.userID;
                mscs.Password = this.password;

                con = new MySqlConnection(mscs.ConnectionString);
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
            /// <param name="paramtypel">資料型態</param>
            public void ParamAdd(String name, Object obj, MySQL.ParamType paramtypel)
            {
                Param p = new Param();
                p.Name = "@" + name;
                p.Obj = obj;
                p.ParamType = paramtypel;

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
                    if (con.State != ConnectionState.Open)
                    {
                        con.Open();
                    }
                    MySqlCommand cmd = new MySqlCommand(commands.ToString(), con);
                    cmd.CommandTimeout = this.cmdTimeoutSecond;
                    cmd.CommandType = CommandType.Text;
                    setParams(ref cmd);

                    MySqlDataAdapter da = new MySqlDataAdapter(cmd);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    dt.TableName = "MySQlDataTable";

                    da.Dispose();
                    cmd.Dispose();
                    con.Close();

                    return dt;
                }
                catch (Exception ex)
                {
                    con.Close();

                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[MySQL]");
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
                    exceptionMessage.AppendLine("[MySQL BeginTransaction]");
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
                    exceptionMessage.AppendLine("[MySQL Commit]");
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
                    exceptionMessage.AppendLine("[MySQL Rollback]");
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
                    MySqlCommand cmd = new MySqlCommand(commands.ToString(), con);
                    cmd.CommandTimeout = this.cmdTimeoutSecond;
                    cmd.CommandType = CommandType.Text;
                    cmd.Transaction = this.trans;

                    setParams(ref cmd);

                    cmd.ExecuteScalar();
                    this.lastInsertedId = cmd.LastInsertedId;
                    cmd.Dispose();
                }
                catch (Exception ex)
                {
                    Rollback();

                    StringBuilder exceptionMessage = new StringBuilder();
                    exceptionMessage.AppendLine("[MySQL]");
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
            private void setParams(ref MySqlCommand cmd)
            {
                cmd.Parameters.Clear();

                foreach (Param p in paramlist)
                {
                    switch (p.ParamType)
                    {
                        case ParamType.DateTime:
                            cmd.Parameters.Add(p.Name, MySqlDbType.DateTime).Value = p.Obj;
                            break;

                        case ParamType.Int32:
                            cmd.Parameters.Add(p.Name, MySqlDbType.Int32).Value = p.Obj;
                            break;

                        case ParamType.String:
                            cmd.Parameters.Add(p.Name, MySqlDbType.String).Value = p.Obj;
                            break;

                        default:
                            break;
                    }
                }
            }
        }
    }
}
```

## SQLite

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
