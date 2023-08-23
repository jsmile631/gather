# Converting string base 64 to byte[]

```code
[SecuritySafeCritical]
[__DynamicallyInvokable]
public unsafe static byte[] FromBase64String(string s)
{
    if (s == null)
    {
        throw new ArgumentNullException("s");
    }

    fixed (char* inputPtr = s)
    {
        return FromBase64CharPtr(inputPtr, s.Length);
    }
}

[SecurityCritical]
private unsafe static byte[] FromBase64CharPtr(char* inputPtr, int inputLength)
{
    while (inputLength > 0)
    {
        int num = inputPtr[inputLength - 1];
        if (num != 32 && num != 10 && num != 13 && num != 9)
        {
            break;
        }

        inputLength--;
    }

    int num2 = FromBase64_ComputeResultLength(inputPtr, inputLength);
    byte[] array = new byte[num2];
    fixed (byte* startDestPtr = array)
    {
        int num3 = FromBase64_Decode(inputPtr, inputLength, startDestPtr, num2);
    }

    return array;
}

[SecurityCritical]
private unsafe static int FromBase64_ComputeResultLength(char* inputPtr, int inputLength)
{
    char* ptr = inputPtr + inputLength;
    int num = inputLength;
    int num2 = 0;
    while (inputPtr < ptr)
    {
        uint num3 = *inputPtr;
        inputPtr++;
        switch (num3)
        {
            case 0u:
            case 1u:
            case 2u:
            case 3u:
            case 4u:
            case 5u:
            case 6u:
            case 7u:
            case 8u:
            case 9u:
            case 10u:
            case 11u:
            case 12u:
            case 13u:
            case 14u:
            case 15u:
            case 16u:
            case 17u:
            case 18u:
            case 19u:
            case 20u:
            case 21u:
            case 22u:
            case 23u:
            case 24u:
            case 25u:
            case 26u:
            case 27u:
            case 28u:
            case 29u:
            case 30u:
            case 31u:
            case 32u:
                num--;
                break;
            case 61u:
                num--;
                num2++;
                break;
        }
    }

    switch (num2)
    {
        case 1:
            num2 = 2;
            break;
        case 2:
            num2 = 1;
            break;
        default:
            throw new FormatException(Environment.GetResourceString("Format_BadBase64Char"));
        case 0:
            break;
    }

    return num / 4 * 3 + num2;
}



[SecurityCritical]
private unsafe static int FromBase64_Decode(char* startInputPtr, int inputLength, byte* startDestPtr, int destLength)
{
  char* ptr = startInputPtr;
  byte* ptr2 = startDestPtr;
  char* ptr3 = ptr + inputLength;
  byte* ptr4 = ptr2 + destLength;
  uint num = 255u;
  while (ptr < ptr3)
  {
      uint num2 = *ptr;
      ptr++;
      if (num2 - 65 <= 25)
      {
          num2 -= 65;
      }
      else if (num2 - 97 <= 25)
      {
          num2 -= 71;
      }
      else if (num2 - 48 <= 9)
      {
          num2 -= 4294967292u;
      }
      else
      {
          if (num2 <= 32)
          {
              if (num2 - 9 <= 1 || num2 == 13 || num2 == 32)
              {
                  continue;
              }

              goto IL_0097;
          }

          if (num2 != 43)
          {
              if (num2 != 47)
              {
                  if (num2 != 61)
                  {
                      goto IL_0097;
                  }

                  if (ptr == ptr3)
                  {
                      num <<= 6;
                      if ((num & 0x80000000u) == 0)
                      {
                          throw new FormatException(Environment.GetResourceString("Format_BadBase64CharArrayLength"));
                      }

                      if ((int)(ptr4 - ptr2) < 2)
                      {
                          return -1;
                      }

                      *(ptr2++) = (byte)(num >> 16);
                      *(ptr2++) = (byte)(num >> 8);
                      num = 255u;
                      break;
                  }

                  for (; ptr < ptr3 - 1; ptr++)
                  {
                      int num3 = *ptr;
                      if (num3 != 32 && num3 != 10 && num3 != 13 && num3 != 9)
                      {
                          break;
                      }
                  }

                  if (ptr == ptr3 - 1 && *ptr == '=')
                  {
                      num <<= 12;
                      if ((num & 0x80000000u) == 0)
                      {
                          throw new FormatException(Environment.GetResourceString("Format_BadBase64CharArrayLength"));
                      }

                      if ((int)(ptr4 - ptr2) < 1)
                      {
                          return -1;
                      }

                      *(ptr2++) = (byte)(num >> 16);
                      num = 255u;
                      break;
                  }

                  throw new FormatException(Environment.GetResourceString("Format_BadBase64Char"));
              }

              num2 = 63u;
          }
          else
          {
              num2 = 62u;
          }
      }

      num = (num << 6) | num2;
      if ((num & 0x80000000u) != 0)
      {
          if ((int)(ptr4 - ptr2) < 3)
          {
              return -1;
          }

          *ptr2 = (byte)(num >> 16);
          ptr2[1] = (byte)(num >> 8);
          ptr2[2] = (byte)num;
          ptr2 += 3;
          num = 255u;
      }

      continue;
  IL_0097:
      throw new FormatException(Environment.GetResourceString("Format_BadBase64Char"));
  }

  if (num != 255)
  {
      throw new FormatException(Environment.GetResourceString("Format_BadBase64CharArrayLength"));
  }

  return (int)(ptr2 - startDestPtr);
}


```
