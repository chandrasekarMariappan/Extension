
 public static class Util
    {

        public static T? ConvertTo<T>(this object data) where T : struct
        {

            return (T)Convert.ChangeType(data, typeof(T));

        }
        public static string[] ConvertToUpper<T>(this string[] data)
        {
            var result = from x in data
                         select x.ToUpper();
            return result.ToArray();
        }
        public static string[] ConvertToLower<T>(this string[] data)
        {
            var result = from x in data
                         select x.ToLower();
            return result.ToArray();
        }
        public static T RemoveLast<T>(this object data, int count)
        {
            if (typeof(T) != typeof(long))
            {
                return (dynamic)data.ToString().Substring(0, (data.ToString().Length - count));
            }
            var dat = Convert.ToString(data);
            return (dynamic)Convert.ToInt64(dat.Substring(0, (dat.Length - count)));
        }
        public static DataTable ConvertToDataTable<T>(this object[] obj) where T : class
        {
            if (obj[0] == null)
                return null;
            var type = obj[0].GetType();
            var dataTable = new DataTable(type.Name);
            var fieldInfo = type.GetFields(BindingFlags.Public | BindingFlags.Instance);
            var propertyInfo = type.GetProperties();
            foreach (var colu in fieldInfo.Select(info => new DataColumn(info.Name, info.FieldType)))
                dataTable.Columns.Add(colu);
            foreach (var colu in propertyInfo.Select(info => new DataColumn(info.Name, info.PropertyType)))
                dataTable.Columns.Add(colu);
            foreach (var item in obj)
            {
                if (((dynamic)item) == null)
                    return dataTable;
                var data = fieldInfo.Select(t => t.GetValue(((dynamic)item))).Cast<dynamic>().ToList();
                data.AddRange(propertyInfo.Select(info => info.GetValue(((dynamic)item), null)).Cast<dynamic>());
                var dr = dataTable.NewRow();
                dr.ItemArray = data.ToArray();
                if (dataTable.Rows != null)
                    dataTable.Rows.Add(dr);
            }
            return dataTable;
        }
        public static T[] ConvertToObject<T>(this DataTable table) where T : class
        {

            var type = typeof(T);
            var output = new T[table.Rows.Count];
            var columnNames = table.Columns.Cast<DataColumn>().Select(x => x.ColumnName).ToArray();
            var fieldInfo = type.GetFields(BindingFlags.Public | BindingFlags.Instance);
            var propertyInfo = type.GetProperties();
            for (int i = 0; i < table.Rows.Count; i++)
            {
                output[i] = (T)Activator.CreateInstance(typeof(T));

                for (int index = 0; index < fieldInfo.Length; index++)
                {

                    var info = fieldInfo[index];
                    if (columnNames.Contains(info.Name))
                    {

                        info.SetValue(output[i], table.Rows[i][info.Name]);
                    }
                }

                for (int index = 0; index < propertyInfo.Length; index++)
                {
                    var info2 = propertyInfo[index];
                    if (columnNames.Contains(info2.Name))
                    {

                        info2.SetValue(output[i], table.Rows[i][info2.Name], null);
                    }
                }
            }

            return output.ToArray();

        }
        public static DataTable NewCopy<T>(this DataTable table) where T : DataTable
        {
            return table.Copy();
        }
        public static T[] Clone<T>(this object[] source) where T : class
        {
            if (!typeof(T).IsSerializable)
            {
                throw new ArgumentException("The type must be serializable.", "source");
            }


            if (Object.ReferenceEquals(source, null))
            {
                return default(T[]);
            }

            IFormatter formatter = new BinaryFormatter();
            Stream stream = new MemoryStream();
            using (stream)
            {
                formatter.Serialize(stream, source);
                stream.Seek(0, SeekOrigin.Begin);
                return (T[])formatter.Deserialize(stream);
            }
        }
        public static DataTable GetDuplicateRecords<T>(this DataTable source, DataTable destination)
        {
            var result = source.AsEnumerable().Except(destination.AsEnumerable(),
                                                    DataRowComparer.Default);
            return result.CopyToDataTable();
        }
        public static DataTable GetSameRecords<T>(this DataTable source, DataTable destination)
        {
            var result = source.AsEnumerable().Intersect(destination.AsEnumerable(),
                                                     DataRowComparer.Default);
            return result.CopyToDataTable();
        }






    }
