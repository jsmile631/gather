# Converting XML to FMD 
sample xml
```
<?xml version="1.0" encoding="UTF-8"?><Fid><Bytes>Rk1SACAyMAABuAAz/v8AAAFlAYgAxQDFAQAAAFZEgK8BGnxkQDQAZGdfQOUAnJ1cgJAA/39bQLQAVD9ZQLwBC5BZQCgAIGRYQMYAqJhYQDoAM2dWQI4An5RVgMoAKDxVQCEAYwhUQGIBSqxUQOIBIZhUgOwAfZ5TgJ8A2ZJTgIMA/HhSQI8ALUZRQDUATQVRQHoASJ5QgIQA3o1QgMEBMnhQQE0BN55QQG0AGFJPgFMAVl5PQHEAf0tPQHoBU01PgI8AMkRMQPcAEjlLQEoAlmhKQP4AukNKgNQBBptKQEsBEDVKQDwAy4JKQNQBL4tKQP8AJ5dJgJgBGX9JgJIBKRtIgE8BKqdIQFMA2zNHgJAAEkVGgPUASDpGQCYA0YdFgG8AzZNEgEYBBZZEQHwBEyVDQGUAeFRCgJgBLyVBgEgA7plBQFEAuow/gA4AR2k+QEoAwZE+gDsA2Is+QGUAcqM9QE0A4po7gHIA0ow7QC4A1yU7gH4A5Xo6QPwA+qA6QB0BHJc5QQIATps4QDcA/jI4gDcAnnw3QF0A70E3QGcA2zg2gRgAJTs2gCEAt3g1QLwBLXc1AAA=</Bytes><Format>1769473</Format><Version>1.0.0</Version></Fid>
```

1. DeserializeXml
```code
public static Fmd DeserializeXml(string data)
{
    using Tracer tracer = new Tracer("Fmd::DeserializeXml");
    if (data == null || data.Length.Equals(0))
    {
        throw new SDKException(Constants.ResultCode.DP_INVALID_PARAMETER, "Xml is invalid and cannot be deserialized.", null);
    }

    try
    {
        XmlDocument xmlDocument = new XmlDocument();
        xmlDocument.LoadXml(data);
        XmlNodeList elementsByTagName = xmlDocument.GetElementsByTagName("Bytes");
        byte[] bytes = Convert.FromBase64String(Fingerbase.GetXmlNodeString(elementsByTagName));
        XmlNodeList elementsByTagName2 = xmlDocument.GetElementsByTagName("Format");
        int format = int.Parse(Fingerbase.GetXmlNodeString(elementsByTagName2));
        XmlNodeList elementsByTagName3 = xmlDocument.GetElementsByTagName("Version");
        string xmlNodeString = Fingerbase.GetXmlNodeString(elementsByTagName3);
        return new Fmd(bytes, format, xmlNodeString);
    }
    catch (Exception ex)
    {
        tracer.Trace(ex.Message);
        throw new SDKException(Constants.ResultCode.DP_INVALID_PARAMETER, ex.Message, ex);
    }
}
```
2. FMD contrutor

```code
public Fmd(byte[] bytes, int format, string version)
    : base(bytes, format, version)
{
    try
    {
        if (format > 2)
        {
            m_header = GetHeader(bytes, format);
            m_views = new List<Fmv>();
            LoadViews();
        }
    }
    catch (Exception)
    {
        throw new SDKException(Constants.ResultCode.DP_FAILURE, string.Empty, null);
    }
}
```

3. GetHeader(bytes, format)
```code
private static DPFJ_Fmd_RECORD_PARAMS GetHeader(byte[] bytes, int format)
{
    int num = 0;
    DPFJ_Fmd_RECORD_PARAMS result = default(DPFJ_Fmd_RECORD_PARAMS);
    int num2 = 0;
    if (1769473 == format)
    {
        result.record_length = Fingerbase.readBytes(bytes, num + 8, 2);
        result.cbeff_id = Fingerbase.readBytes(bytes, num + 10, 4);
        num2 = 2;
        if (result.record_length == 0)
        {
            result.record_length = Fingerbase.readBytes(bytes, num + 10, 4);
            result.cbeff_id = Fingerbase.readBytes(bytes, num + 14, 4);
            num2 = 6;
        }
    }
    else
    {
        if (16842753 != format)
        {
            return result;
        }

        result.record_length = Fingerbase.readBytes(bytes, num + 8, 4);
        result.cbeff_id = 0;
    }

    result.capture_equipment_comp = Fingerbase.readBytes(bytes, num + 12 + num2, 1) & 0xF0;
    result.capture_equipment_id = Fingerbase.readBytes(bytes, num + 12 + num2, 2) & 0xFFF;
    result.width = Fingerbase.readBytes(bytes, num + 14 + num2, 2);
    result.height = Fingerbase.readBytes(bytes, num + 16 + num2, 2);
    result.resolution = Fingerbase.readBytes(bytes, num + 18 + num2, 2);
    result.view_cnt = Fingerbase.readBytes(bytes, num + 22 + num2, 1);
    return result;
}
```

4. LoadViews()
```code
private void LoadViews()
{
    int num = 0;
    for (int i = 0; i < m_header.view_cnt; i++)
    {
        num = GetViewOffset(m_bytes, m_format, i);
        DPFJ_Fmd_VIEW_PARAMS viewHeader = GetViewHeader(m_bytes, m_format, num);
        m_views.Add(new Fmv(GetView(m_bytes, num, viewHeader.minutia_cnt, viewHeader.ext_block_length)));
    }
}

private static DPFJ_Fmd_VIEW_PARAMS GetViewHeader(byte[] bytes, int format, int offset)
{
    DPFJ_Fmd_VIEW_PARAMS result = default(DPFJ_Fmd_VIEW_PARAMS);
    int num = 0;
    int num2 = num + offset;
    result.finger_position = Fingerbase.readBytes(bytes, num2, 1);
    result.view_number = (Fingerbase.readBytes(bytes, num2 + 1, 1) >> 4) & 0xF;
    result.impression_type = Fingerbase.readBytes(bytes, num2 + 1, 1) & 0xF;
    result.quality = Fingerbase.readBytes(bytes, num2 + 2, 1);
    result.minutia_cnt = Fingerbase.readBytes(bytes, num2 + 3, 1);
    result.ext_block_length = Fingerbase.readBytes(bytes, num2 + 4 + result.minutia_cnt * DPFJ_Fmd_ANSI_ISO_MINITIA_LENGTH, 2);
    result.ext_block = new byte[result.ext_block_length];
    Buffer.BlockCopy(bytes, num2 + 4 + result.minutia_cnt * DPFJ_Fmd_ANSI_ISO_MINITIA_LENGTH + 2, result.ext_block, 0, result.ext_block_length);
    return result;
}

private static byte[] GetView(byte[] bytes, int offset, int minutiaeCount, int extBlockLength)
{
    byte[] array = new byte[DPFJ_Fmd_ANSI_ISO_VIEW_HEADER_LENGTH + minutiaeCount * DPFJ_Fmd_ANSI_ISO_MINITIA_LENGTH + extBlockLength + 2];
    Buffer.BlockCopy(bytes, offset, array, 0, array.Length);
    return array;
}
```
