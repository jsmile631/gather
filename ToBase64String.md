# Converting byte[] to string base 64

```code
[__DynamicallyInvokable]
public static string ToBase64String(byte[] inArray)
{
    if (inArray == null)
    {
        throw new ArgumentNullException("inArray");
    }

    return ToBase64String(inArray, 0, inArray.Length, Base64FormattingOptions.None);
}

[SecuritySafeCritical]
[ComVisible(false)]
public unsafe static string ToBase64String(byte[] inArray, int offset, int length, Base64FormattingOptions options)
{
    if (inArray == null)
    {
        throw new ArgumentNullException("inArray");
    }

    if (length < 0)
    {
        throw new ArgumentOutOfRangeException("length", Environment.GetResourceString("ArgumentOutOfRange_Index"));
    }

    if (offset < 0)
    {
        throw new ArgumentOutOfRangeException("offset", Environment.GetResourceString("ArgumentOutOfRange_GenericPositive"));
    }

    if (options < Base64FormattingOptions.None || options > Base64FormattingOptions.InsertLineBreaks)
    {
        throw new ArgumentException(Environment.GetResourceString("Arg_EnumIllegalVal", (int)options));
    }

    int num = inArray.Length;
    if (offset > num - length)
    {
        throw new ArgumentOutOfRangeException("offset", Environment.GetResourceString("ArgumentOutOfRange_OffsetLength"));
    }

    if (num == 0)
    {
        return string.Empty;
    }

    bool insertLineBreaks = options == Base64FormattingOptions.InsertLineBreaks;
    int length2 = ToBase64_CalculateAndValidateOutputLength(length, insertLineBreaks);
    string text = string.FastAllocateString(length2);
    fixed (char* outChars = text)
    {
        fixed (byte* inData = inArray)
        {
            int num2 = ConvertToBase64Array(outChars, inData, offset, length, insertLineBreaks);
            return text;
        }
    }
}

private static int ToBase64_CalculateAndValidateOutputLength(int inputLength, bool insertLineBreaks)
{
    long num = (long)inputLength / 3L * 4;
    num += ((inputLength % 3 != 0) ? 4 : 0);
    if (num == 0L)
    {
        return 0;
    }

    if (insertLineBreaks)
    {
        long num2 = num / 76;
        if (num % 76 == 0L)
        {
            num2--;
        }

        num += num2 * 2;
    }

    if (num > int.MaxValue)
    {
        throw new OutOfMemoryException();
    }

    return (int)num;
}

[SecurityCritical]
private unsafe static int ConvertToBase64Array(char* outChars, byte* inData, int offset, int length, bool insertLineBreaks)
{
    int num = length % 3;
    int num2 = offset + (length - num);
    int num3 = 0;
    int num4 = 0;
    fixed (char* ptr = base64Table)
    {
        int i;
        for (i = offset; i < num2; i += 3)
        {
            if (insertLineBreaks)
            {
                if (num4 == 76)
                {
                    outChars[num3++] = '\r';
                    outChars[num3++] = '\n';
                    num4 = 0;
                }

                num4 += 4;
            }

            outChars[num3] = ptr[(inData[i] & 0xFC) >> 2];
            outChars[num3 + 1] = ptr[((inData[i] & 3) << 4) | ((inData[i + 1] & 0xF0) >> 4)];
            outChars[num3 + 2] = ptr[((inData[i + 1] & 0xF) << 2) | ((inData[i + 2] & 0xC0) >> 6)];
            outChars[num3 + 3] = ptr[inData[i + 2] & 0x3F];
            num3 += 4;
        }

        i = num2;
        if (insertLineBreaks && num != 0 && num4 == 76)
        {
            outChars[num3++] = '\r';
            outChars[num3++] = '\n';
        }

        switch (num)
        {
            case 2:
                outChars[num3] = ptr[(inData[i] & 0xFC) >> 2];
                outChars[num3 + 1] = ptr[((inData[i] & 3) << 4) | ((inData[i + 1] & 0xF0) >> 4)];
                outChars[num3 + 2] = ptr[(inData[i + 1] & 0xF) << 2];
                outChars[num3 + 3] = ptr[64];
                num3 += 4;
                break;
            case 1:
                outChars[num3] = ptr[(inData[i] & 0xFC) >> 2];
                outChars[num3 + 1] = ptr[(inData[i] & 3) << 4];
                outChars[num3 + 2] = ptr[64];
                outChars[num3 + 3] = ptr[64];
                num3 += 4;
                break;
        }
    }

    return num3;
}
```
