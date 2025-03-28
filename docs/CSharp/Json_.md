# JSON序列化與序列化

## 使用System.Web.Script.Serialization

```csharp
using System;
using System.Web.Script.Serialization;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //建立員工資料
            Employee emp = new Employee();
            emp.EmpID = "A1234";
            emp.EmpName = "王小明";
            emp.EmpBirthday = DateTime.Now.AddYears(-25);

            Json json = new Json();

            //序列化
            String jsonStr = json.Serialize(emp);
            Console.WriteLine(jsonStr);

            //反序列化
            Employee newemp = json.Deserialize<Employee>(jsonStr);
            Console.WriteLine("EmpID = {0}", newemp.EmpID);
        }

        /// <summary>
        /// 員工資料 Class
        /// </summary>
        public class Employee
        {
            public String EmpID { get; set; }
            public String EmpName { get; set; }
            public DateTime EmpBirthday { get; set; }
        }

        /// <summary>
        /// JSON
        /// </summary>
        public class Json
        {
            /// <summary>
            /// 序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="obj"></param>
            /// <returns></returns>
            public string Serialize<T>(T obj)
            {
                JavaScriptSerializer js = new JavaScriptSerializer();
                string json = js.Serialize(obj); // 序列化
                return json;
            }

            /// <summary>
            /// 反序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="jsonstr"></param>
            /// <returns></returns>
            public T Deserialize<T>(String jsonstr)
            {
                JavaScriptSerializer js = new JavaScriptSerializer();
                T obj = js.Deserialize<T>(jsonstr); // 反序列化
                return obj;
            }
        }
    }
}
```

## 使用System.Runtime.Serialization.Json


```csharp
using System;
using System.IO;
using System.Text;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Json;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //建立員工資料
            Employee emp = new Employee();
            emp.EmpID = "A1234";
            emp.EmpName = "王小明";
            emp.EmpBirthday = DateTime.Now.AddYears(-25);

            Json json = new Json();

            //序列化
            String jsonStr = json.Serialize(emp);
            Console.WriteLine(jsonStr);

            //反序列化
            Employee newemp = json.Deserialize<Employee>(jsonStr);
            Console.WriteLine("EmpID = {0}", newemp.EmpID);
        }

        /// <summary>
        /// 員工資料 Class
        /// </summary>
        [DataContract]
        public class Employee
        {
            [DataMember]
            public String EmpID { get; set; }
            [DataMember]
            public String EmpName { get; set; }
            [DataMember]
            public DateTime EmpBirthday { get; set; }
        }

        /// <summary>
        /// JSON
        /// </summary>
        public class Json
        {
            /// <summary>
            /// 序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="obj"></param>
            /// <returns></returns>
            public string Serialize<T>(T obj)
            {
                DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(T));
                using (MemoryStream stream = new MemoryStream())
                {
                    serializer.WriteObject(stream, obj);
                    return Encoding.UTF8.GetString(stream.ToArray());
                }
            }

            /// <summary>
            /// 反序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="jsonstr"></param>
            /// <returns></returns>
            public T Deserialize<T>(String jsonstr)
            {
                DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(T));
                using (MemoryStream stream = new MemoryStream(Encoding.UTF8.GetBytes(jsonstr)))
                {
                    return (T)serializer.ReadObject(stream);
                }
            }
        }
    }
}
```

## 使用Newtonsoft.Json

```csharp
using System;
using Newtonsoft.Json;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //建立員工資料
            Employee emp = new Employee();
            emp.EmpID = "A1234";
            emp.EmpName = "王小明";
            emp.EmpBirthday = DateTime.Now.AddYears(-25);

            Json json = new Json();

            //序列化
            String jsonStr = json.Serialize(emp);
            Console.WriteLine(jsonStr);

            ////反序列化
            Employee newemp = json.Deserialize<Employee>(jsonStr);
            Console.WriteLine("EmpName = {0}", newemp.EmpName);
        }

        /// <summary>
        /// 員工資料 Class
        /// </summary>
        public class Employee
        {
            public String EmpID { get; set; }
            public String EmpName { get; set; }
            public DateTime EmpBirthday { get; set; }
        }

        /// <summary>
        /// JSON
        /// </summary>
        public class Json
        {
            /// <summary>
            /// 序列化
            /// </summary>
            /// <param name="obj"></param>
            /// <returns></returns>
            public string Serialize(Object obj)
            {
                return JsonConvert.SerializeObject(obj);
            }

            /// <summary>
            /// 反序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="jsonstr"></param>
            /// <returns></returns>
            public T Deserialize<T>(String jsonstr)
            {
                return JsonConvert.DeserializeObject<T>(jsonstr);
            }
        }
    }
}
```

## 使用System.Text.Json

```csharp
using System;
using System.Text.Json;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //建立員工資料
            Employee emp = new Employee();
            emp.EmpID = "A1234";
            emp.EmpName = "王小明";
            emp.EmpBirthday = DateTime.Now.AddYears(-25);

            Json json = new Json();

            //序列化
            String jsonStr = json.Serialize(emp);
            Console.WriteLine(jsonStr);

            ////反序列化
            Employee newemp = json.Deserialize<Employee>(jsonStr);
            Console.WriteLine("EmpName = {0}", newemp.EmpName);
        }

        /// <summary>
        /// 員工資料 Class
        /// </summary>
        public class Employee
        {
            public String EmpID { get; set; }
            public String EmpName { get; set; }
            public DateTime EmpBirthday { get; set; }
        }

        /// <summary>
        /// JSON
        /// </summary>
        public class Json
        {
            /// <summary>
            /// 序列化
            /// </summary>
            /// <param name="obj"></param>
            /// <returns></returns>
            public string Serialize(Object obj)
            {
                return JsonSerializer.Serialize(obj);
            }

            /// <summary>
            /// 反序列化
            /// </summary>
            /// <typeparam name="T"></typeparam>
            /// <param name="jsonstr"></param>
            /// <returns></returns>
            public T Deserialize<T>(String jsonstr)
            {
                return JsonSerializer.Deserialize<T>(jsonstr);
            }
        }
    }
}
```
