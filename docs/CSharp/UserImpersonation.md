# 模擬執行身分

在程式執行中，切換執行的身分

```csharp
/// <summary>
/// 模擬執行身分
/// </summary>
public class UserImpersonation
{
    [DllImport("advapi32.dll", SetLastError = true)]
    public static extern bool LogonUser(string lpszUsername, string lpszDomain, string lpszPassword, int dwLogonType, int dwLogonProvider, out IntPtr phToken);

    private WindowsImpersonationContext ic;

    /// <summary>
    /// 模擬執行身分
    /// </summary>
    /// <param name="Domain"></param>
    /// <param name="UserName"></param>
    /// <param name="Password"></param>
    public void Impersonate(String Domain, String UserName, String Password)
    {
        IntPtr userToken;
        LogonUser(UserName, Domain, Password, 9, 0, out userToken);
        this.ic = WindowsIdentity.Impersonate(userToken);
    }

    /// <summary>
    /// 還原執行身分
    /// </summary>
    public void Revert()
    {
        this.ic.Undo();
    }
}
```