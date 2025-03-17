# MySql/MariaDB

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
            private MySqlConnection com;

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

                com = new MySqlConnection(mscs.ConnectionString);
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
            /// 添加指令
            /// </summary>
            /// <param name="command"></param>
            public void CommandAdd(String command)
            {
                commands.AppendLine(command);
            }

            /// <summary>
            ///  添加指令
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
            /// <param name="name"></param>
            /// <param name="obj"></param>
            /// <param name="paramtypel"></param>
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
                    if (com.State != ConnectionState.Open)
                    {
                        com.Open();
                    }
                    MySqlCommand cmd = new MySqlCommand(commands.ToString(), com);
                    cmd.CommandTimeout = this.cmdTimeoutSecond;
                    cmd.CommandType = CommandType.Text;

                    setParams(ref cmd);

                    MySqlDataAdapter da = new MySqlDataAdapter(cmd);
                    DataTable dt = new DataTable();
                    da.Fill(dt);
                    dt.TableName = "MySQlDataTable";

                    da.Dispose();
                    cmd.Dispose();
                    com.Close();

                    return dt;
                }
                catch (Exception ex)
                {
                    com.Close();

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
                    if (com.State != ConnectionState.Open)
                    {
                        com.Open();
                    }
                    this.trans = com.BeginTransaction();
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
                    com.Clone();
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
            public void Rollback()
            {
                try
                {
                    trans.Rollback();
                    com.Clone();
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
            /// <returns></returns>
            public void NonQuery()
            {
                try
                {
                    MySqlCommand cmd = new MySqlCommand(commands.ToString(), com);
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
