# Json to DB dynamically

This is a c# code which takes a Json string and creates a db table out of it under master db dynamically. See below code:


 -----------------------------------------------------------------------
 
 
using System;
using System.Collections.Generic;
using System.Diagnostics;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
 
using System.Data.SqlClient;
using System.Data;
using System.Linq;

namespace JsonToDb.Pages
{
   

    public class IndexModel : PageModel
    {
        //table name
        public  static string fileName = "Tabel1";

        string strMsg = System.IO.File.ReadAllText(@"C:\Users\DEV009.SERVER\source\repos\JsonToDb\JsonToDb\test.json");

        string createTable = string.Empty;

        public void OnGet()
         {
            processJsonAndCreateSqlTable();
            insertData();

        }


        public void processJsonAndCreateSqlTable()
        {
             
            string createTableFields = string.Empty;
        

            if (ValidateJSON(strMsg))
            {

                string json = strMsg;


                List<String> fields = new List<String>();

                JArray obj = Newtonsoft.Json.JsonConvert.DeserializeObject<JArray>(json);
                foreach (var result in obj)
                {

                    Console.WriteLine(result.ToString());


                    JObject jsonObject = JObject.Parse(result.ToString());



                    IEnumerable<JToken> jTokens = jsonObject.Descendants().Where(p => p.Count() == 0);
                    Dictionary<string, string> dict = jTokens.Aggregate(new Dictionary<string, string>(), (properties, jToken) =>
                    {
                        properties.Add(jToken.Path, jToken.ToString());
                        return properties;
                    });

                    List<string> collectFileds = new List<string>();



                    foreach (var kv in dict)
                    {
                        collectFileds.Add(kv.Key);
                    }

                 fields = collectFileds.Distinct().ToList();

                  
                }


                    foreach (var foo in fields)
                    {
                        createTableFields += "\"" + foo + "\"" + " varchar(max), ";

                    }

                
      


                //create db 

                // your connection string
                string connectionString = "Server=SERVER\\SQLEXPRESS;Integrated Security=True;";

                // your query:
                 var commandStr = "CREATE TABLE " + fileName + "(" + createTableFields + ")";

                var conn = new SqlConnection(connectionString);
                var command = new SqlCommand(commandStr, conn);
               

                try
                {
                    conn.Open();
                    command.ExecuteNonQuery();
                    Console.WriteLine("Table is created successfully in master");

                 
                }
                catch (Exception ex)
                {
                    Console.WriteLine(ex.ToString());
                }
                finally
                {
                    if ((conn.State == ConnectionState.Open))
                    {
                        conn.Close();
                    }
                }





            }
            else
            {

                Console.Write("Json not valid");
            }


        }



        public void insertData()
        {
            
            string createTableFields = string.Empty;
            string createTableValues = string.Empty;



            if (ValidateJSON(strMsg))
            {
                string json = strMsg;
                //insert data into table created
                
                //create table within the db once db is created
                int loopcounter = 0;

                JArray obj = Newtonsoft.Json.JsonConvert.DeserializeObject<JArray>(json);
                foreach (var result in obj)
                {



                   

                    Console.WriteLine(result.ToString());


                    JObject jsonObject = JObject.Parse(result.ToString());

                    IEnumerable<JToken> jTokens = jsonObject.Descendants().Where(p => p.Count() == 0);
                    Dictionary<string, string> dict = jTokens.Aggregate(new Dictionary<string, string>(), (properties, jToken) =>
                    {
                        properties.Add(jToken.Path, jToken.ToString());
                        return properties;
                    });

                    createTableValues += "INSERT INTO " + fileName + " (";

                    foreach (var kv in dict)
                    {

                        Console.WriteLine(kv.Key + ":" + kv.Value);

                        List<String> fields = new List<String>();
                        List<String> values = new List<String>();

                        fields.Add(kv.Key);
                        values.Add(kv.Value);

                        int dictCount = dict.Count;


                        foreach (var foo in fields)
                        {
                            loopcounter = loopcounter + 1;

                            createTableValues +=  '"' + foo + '"' ;
                            if (loopcounter < dict.Count)
                            {
                                createTableValues += ", ";
                            }
                        }

                    }
                    loopcounter = 0;

                    createTableValues += " ) VALUES (";


                    foreach (var kv in dict)
                    {
                  
                        Console.WriteLine(kv.Key + ":" + kv.Value);

                        List<String> fields = new List<String>();
                        List<String> values = new List<String>();

                        fields.Add(kv.Key);
                        values.Add(kv.Value);

                        int dictCount = dict.Count;
                     

                        foreach (var foo in values)
                        {
                            loopcounter = loopcounter + 1;

                            createTableValues += "'" + foo + "'";
                            if (loopcounter < dict.Count)
                            {
                                createTableValues += ", ";
                            }
                        }

                    }
                    loopcounter = 0;
                    //close
                    createTableValues += ");";

                }
                    

                
                //create db 

                // your connection string
                string connectionString = "Server=SERVER\\SQLEXPRESS;Integrated Security=True;";

                // your query:

               var commandStr = createTableValues;

                var conn = new SqlConnection(connectionString);
                var command = new SqlCommand(commandStr, conn);


                try
                {
                    conn.Open();
                    command.ExecuteNonQuery();
                    Console.WriteLine("Data" + createTableValues + " added to the table");


                }
                 catch (Exception ex)
                {
                    Console.WriteLine(ex.ToString());
                }
                finally
                {
                    if ((conn.State == ConnectionState.Open))
                    {
                        conn.Close();
                    }
                }





            }


        }

 


        public bool ValidateJSON(string s)
        {
            try
            {
                JToken.Parse(s);
                return true;
            }
            catch (JsonReaderException ex)
            {
                Trace.WriteLine(ex);
                return false;
            }
        }

 
        static string GetDbCreationQuery()
        {

            // your db name
            string dbName = fileName;

            // db creation query
            string query = "CREATE DATABASE " + dbName + ";";

            return query;
        }


    }
}

-------------------------------------

Sample Json file:


[
  {
    "menu": {
      "id": "file",
      "value": "File",
      "popup": {
        "menuitem": [
          {
            "value": "New",
            "onclick": "CreateNewDoc()"
          },
          {
            "value": "Open",
            "onclick": "OpenDoc()"
          },
          {
            "value": "Close",
            "onclick": "CloseDoc()",
            "onShutDown": "ShutProg()"
          }
        ]
      }
    }
  },
  {
    "menu": {
      "id": "file",
      "value": "File",
      "popup": {
        "menuitem": [
          {
            "value": "New",
            "onclick": "CreateNewDoc()"
          },
          {
            "value": "Open",
            "onclick": "OpenDoc()"
          },
          {
            "value": "Close",
            "onclick": "CloseDoc()",
            "onShutDown": "ShutProg()",
            "onLoad": "LoadProg()"
          }
        ]
      }
    }
  }
]

