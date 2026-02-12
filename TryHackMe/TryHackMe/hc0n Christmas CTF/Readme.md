<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; line-height: 1.6; color: #24292e; max-width: 800px; margin: 0 auto; padding: 20px; }
        h1, h2 { border-bottom: 1px solid #eaecef; padding-bottom: 0.3em; }
        code { background-color: rgba(27,31,35,0.05); border-radius: 3px; padding: 0.2em 0.4em; font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace; }
        pre { background-color: #f6f8fa; padding: 16px; border-radius: 6px; overflow: auto; }
        pre code { background: none; padding: 0; }
        blockquote { border-left: 0.25em solid #dfe2e5; color: #6a737d; padding: 0 1em; margin: 0; }
        table { border-collapse: collapse; width: 100%; margin: 16px 0; }
        table th, table td { border: 1px solid #dfe2e5; padding: 6px 13px; }
        table tr:nth-child(even) { background-color: #f6f8fa; }
        .badge { display: inline-block; padding: 2px 10px; border-radius: 10px; color: white; font-weight: bold; font-size: 12px; }
        .hard { background-color: #d73a49; }
    </style>
</head>
<body>

    <h1>hc0n Christmas CTF <span class="badge hard">Level: Hard</span></h1>
    <p>This writeup covers the initial foothold and exploitation phase for the hc0n Christmas CTF.</p>

    <img src="Capture.PNG" alt="Main Challenge Banner" style="max-width:100%;">

    <h2>📌 Initial Setup</h2>
    <p>The target IP address is stored as a variable for the session:</p>
    <pre><code>domain=$target_ip</code></pre>

    <hr>

    <h2>🕵️ Phase One: Reconnaissance & Scanning</h2>
    <p>I used <strong>Rustscan</strong> for rapid discovery, leveraging the Nmap engine for service detection.</p>

    <pre><code>rustscan -a $domain --ulimit 5000 -- -sC -sV -T4</code></pre>

    <table>
        <thead>
            <tr><th>Port</th><th>State</th><th>Service</th><th>Version</th></tr>
        </thead>
        <tbody>
            <tr><td>22</td><td>Open</td><td>SSH</td><td>OpenSSH 7.2p2 (Ubuntu)</td></tr>
            <tr><td>80</td><td>Open</td><td>HTTP</td><td>Apache httpd 2.4.18</td></tr>
            <tr><td>8080</td><td>Open</td><td>HTTP-Proxy</td><td>Apache (Potential redirector)</td></tr>
        </tbody>
    </table>

    <hr>

    <h2>🌐 Web Enumeration</h2>
    <h3>1. Port 8080</h3>
    <p>Visiting the proxy port reveals an encrypted string:</p>
    <code>RwO9+7tuGJ3nc1cIhN4E31WV/qeYGLURrcS7K+Af85w=</code>

    <h3>2. The Cicada 3301 Hint</h3>
    <p>The <code>robots.txt</code> file led to <code>iv.png</code>. Using a Cicada 3301 alphabet map, the decoded string is:</p>
    <blockquote><strong>THEIVFORINGEOAEY</strong></blockquote>
    <p>This is our <strong>Initialization Vector (IV)</strong> for AES-CBC.</p>

    <hr>

    <h2>🔑 Credential Hunting</h2>
    <p>By checking the <code>/1</code> subdirectory and switching the HTTP verb to <code>OPTIONS</code> via Burp Suite, we obtained the first half of the password: <code>Gf7MRr55</code>.</p>
    <p>The second half was found inside the <code>hola</code> binary strings:</p>
    <pre><code>strings hola | grep "n$@#PDuliL"</code></pre>
    <p><strong>Full SSH Password:</strong> <code>Gf7MRr55n$@#PDuliL</code></p>

    <hr>

    <h2>🏁 Final Foothold</h2>
    <p>After reverse engineering the <code>app-release.apk</code> with <strong>Jadx-GUI</strong>, we identified the SecretKeySpec as <code>hconkwithyhackme</code>.</p>

    <ul>
        <li><strong>IV:</strong> THEIVFORINGEOAEY</li>
        <li><strong>Key:</strong> hconkwithyhackme</li>
        <li><strong>Cipher:</strong> AES-CBC-PKCS5Padding</li>
    </ul>

    <p>Decryption revealed the username: <code>thedarktangent</code>.</p>

    <pre><code>ssh thedarktangent@$domain</code></pre>
    <img src="flag.png" alt="Success Flag" style="max-width:100%;">

    <hr>

    <h2>🎯 Key Learnings</h2>
    <ul>
        <li>Chaining cryptographic findings from multiple sources.</li>
        <li>Utilizing <strong>Oracle Padding Attacks</strong> with Padbuster to forge administrative cookies.</li>
        <li>Reverse engineering Android binaries to extract hardcoded encryption keys.</li>
    </ul>

</body>
</html>
