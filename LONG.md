# MailFS – Total Commander WFX Plugin

IMAP/POP3 e-mail access as a virtual filesystem in Total Commander.

---

## Overview

MailFS is a WFX plugin (filesystem plugin) for Total Commander 64-bit that displays
configured e-mail accounts as folders in TC's network environment.

- **Root level:** One folder per configured account
- **Account level:** The server's real IMAP mailbox hierarchy
- **Folder level:** Messages as virtual `.eml` files
- **Download:** Export messages as complete EML files by copying them out of TC
- **Sending:** Full SMTP send support (see "Sending a message" below)

---

## Requirements

| Component | Version |
|---|---|
| Total Commander | 10.0+ (64-bit) |
| Windows | 10 / 11 (x64) |
| OpenSSL | 3.x (libssl-3-x64.dll, libcrypto-3-x64.dll) |

OpenSSL 3.x for Windows 64-bit: https://slproweb.com/products/Win32OpenSSL.html
(the "Light" version is enough; only the two DLLs are needed.)

---

## Installation

1. Copy the following files into a folder of your choice (e.g. `C:\TC\Plugins\mailfs\`):
   - `mailfs.wfx`
   - `libssl-3-x64.dll`
   - `libcrypto-3-x64.dll`
   - the `locale\` folder (language files; without this folder the plugin runs in English)

2. In Total Commander: **Configuration → Options → Plugins → File system plugins → Add**

3. Select `mailfs.wfx` and confirm.

4. The plugin now appears in TC's network environment (left panel: `\` → Network → MailFS).

---

## Configuring an account

**Method 1 – via the TC menu:**
Navigate to the MailFS plugin in the left panel → **Alt+Enter** or **Files → Properties**.

**Method 2 – directly:**
Right-click the MailFS icon in TC's network area → **Properties**.

In the **MailFS Accounts** dialog:
1. Click **New...**
2. Enter the e-mail address → the provider is detected automatically
3. Click **Auto-configure** → server/port get pre-filled
4. Choose authentication:
   - **OAuth2:** "Sign in via browser..." → a browser opens → the token is saved after sign-in
   - **Password:** "Enter password..." → stored securely in the Windows Credential Manager
5. **Test connection** → checks the connection to the IMAP server
6. Optional (IMAP only): **Edit folders...** → fetches the folder list from the
   server and lets you choose which folders are shown in the TC panel
   (click to toggle a folder, no Ctrl/Shift needed). If all folders are
   selected, newly created server-side folders will automatically show up
   too in the future; if only a subset is selected, it stays exactly at
   that selection.
7. Optional (for sending mail): **SMTP...** → set SMTP server/port/encryption
   (via **Auto-configure**, same as for receiving), plus the from address and
   **Save copy to Sent folder** (default: off, only takes effect for IMAP).
   Username/password/OAuth2 are taken from the configured receiving account,
   not asked for separately.
8. **OK**

---

## Supported providers

| Provider | Auto-detected | OAuth2 | Password |
|---|---|---|---|
| Gmail | ✓ | ✓ (recommended) | ✓ (app password needed) |
| Outlook / Microsoft 365 | ✓ | ✓ (recommended) | ✓ |
| Yahoo | ✓ | ✓ | ✓ (app password) |
| GMX | ✓ | – | ✓ |
| WEB.DE | ✓ | – | ✓ |
| iCloud | ✓ | – | ✓ (app password) |
| Custom | – | Optional | ✓ |

---

## Usage

### Viewing messages
Navigate to the account in the TC panel → open an IMAP folder → messages appear as `.eml` files.

### Downloading / opening a message
- **F5 (Copy):** copy the message as a `.eml` file to a local drive
- Then open it with an installed EML viewer (e.g. Outlook, Thunderbird, or a TC viewer plugin)

### Setting up Custom Columns
1. In a mail folder: **View → Configure custom columns**
2. New profile → **Add plugin columns** → select **mailfs**
3. Available fields: **From, To, Subject, Received date, Size, Attachment,
   Flags, Status, TotalMails, NewMails**
4. Alternatively: the plugin suggests a default layout via `FsContentGetDefaultView`.

**Status** (`<fs>.Status`) — per message: `new` if marked unread on the
server, `old` if read, `---` if unknown (e.g. POP3, which has no concept of
read status). The value follows the UI language.

**TotalMails** / **NewMails** (`<fs>.TotalMails`, `<fs>.NewMails`) — per
**folder** row (e.g. when viewing an account's folder list): total count
resp. count of unread messages in that folder (determined via IMAP
STATUS). Always `---` for message rows, the account itself, and POP3.

### Testing an account
In the **MailFS Accounts** dialog → select account → **Test**

### Sending a message
The account folder shows an entry **"*** New mail ***"** at the bottom
(localized). Opening it (Enter/double-click) starts the compose window:
To/Cc/Bcc (comma- or semicolon-separated), Subject, body text, plus
**Attach...** (click repeatedly to select multiple files). **Send**
connects to the configured SMTP server, authenticates with the receiving
account's credentials, and sends the message; if **"Save copy to Sent
folder"** is enabled (IMAP only), a copy is additionally written to the
detected "Sent" folder (a failure there is only logged, the send itself
still counts as successful). SMTP must be set up beforehand in the account
dialog via **SMTP...**.

**Attaching file(s) via copy/drag:** Copying one or more files into an
account via F5 or drag & drop – into any folder – automatically opens the
compose window with the file(s) already attached. The original file is
neither modified nor deleted (not even on "Move"); MailFS makes its own
internal copy.

### Address book
Next to To/Cc/Bcc in the compose window, the **"AB"** button opens the
address book (not split per account – one shared address book for all
accounts). The list shows every saved address with two checkboxes:
**Favorite** (never removed automatically) and **Default for To/Cc/Bcc**
(automatically filled into the respective field when a new compose window
opens – stored separately per field). Select multiple addresses by
clicking; toggling either checkbox applies to all marked rows.
Double-clicking an address inserts just that one into the field whose
"AB" button opened the address book; **Apply** inserts all marked
addresses, comma-separated. **Remove** permanently deletes an address
(favorites included).

After every successful send, all recipients (To/Cc/Bcc) are automatically
recorded in the address book. On save, all favorites plus the most
recently used `MaxAddresses` non-favorites are kept; older non-favorites
fall out automatically.

### Replying via F7 ("new folder")
In a message list, press **F7** and, as the name for the new folder, enter
exactly the filename of an existing message **without** the `.eml`
extension (e.g. `20260611_0638_Badminton News 6_26_UID102017638` for the
file `...UID102017638.eml`). **No** folder gets created – instead, the
compose window opens as a reply to exactly that message: subject with a
`RE: ` prefix, To/Cc taken from the original mail, in the body two blank
lines, a line with `---`, below that the quoted original text, plus
correctly set `In-Reply-To`/`References` headers for thread association on
the recipient's side. Works for both IMAP and POP3 accounts. Any other
folder name (that doesn't match an existing message) fails as usual, since
actually creating folders isn't supported.

---

## Language

MailFS is localized (German, English, Russian, Ukrainian). In the
**MailFS Accounts** dialog, the **Language** combo box lets you pick a
fixed language; **Automatic (system language)** follows Windows' UI
language. If the detected language isn't available, English is used
automatically.

The language files live in the `locale\` folder next to `mailfs.wfx`
(`mailfs.de.po`, `mailfs.en.po`, `mailfs.ru.po`, `mailfs.uk.po`) and can be
adjusted or extended with a gettext/PO editor (e.g. Poedit). English is
the language compiled into the plugin by default and needs no `.po` file.

The choice is stored in `mailfs.ini` (see below) and takes effect
immediately – the dialog reopens automatically on a language change.

---

## Configuration files

### mailfs.ini
Located in the same folder as `mailfs.wfx`.
Contains **only non-sensitive** configuration (server, port, encryption etc.).
**No passwords or tokens** are stored here.

Example:
```ini
[Global]
Version=1
AccountCount=1
Language=auto
Logging=0

[Account0]
ID=a1b2c3d4-e5f6-7890-abcd-ef1234567890
DisplayName=My Gmail
Email=user@gmail.com
Provider=1
Active=True
Protocol=0
ImapServer=imap.gmail.com
ImapPort=993
ImapEncryption=1
ImapUsername=user@gmail.com
ImapAuth=1
Pop3Server=pop.gmail.com
Pop3Port=995
Pop3Encryption=1
Pop3Username=user@gmail.com
Pop3Auth=1
VisibleFolders=INBOX|INBOX.Sent
SmtpServer=smtp.gmail.com
SmtpPort=587
SmtpEnc=2
SmtpFrom=user@gmail.com
SmtpSaveToSent=False
```

`Logging` (`[Global]`, default `0`) enables/disables the debug log written
to `%TEMP%\mailfs_debug.log`. Set to `1` and restart/reconnect the plugin to
turn logging on; there is no in-UI toggle for this yet.

`VisibleFolders` is set via **Edit folders...** in the account dialog
(a `|`-separated list of folder paths). If the key is missing or empty,
all of the account's folders are shown.

`SmtpServer`/`SmtpPort`/`SmtpEnc` (0=Auto, 1=TLS, 2=STARTTLS, 3=none) and
`SmtpFrom` are set via **SMTP...** in the account dialog; an empty
`SmtpFrom` means "use Email". `SmtpSaveToSent` (default `False`) controls
whether an additional copy is written to the "Sent" folder via IMAP APPEND
when sending. The username/password/OAuth2 tokens used for sending are
**not** stored separately, but taken from whichever receive protocol
(IMAP/POP3) is active.

The address book (see "Address book" above) lives in its own section,
`[Addresses]`, independent of any account:
```ini
[Addresses]
MaxAddresses=10
Count=2
Address0=boss@company.com
Address0.Fav=1
Address0.DefTo=0
Address0.DefCc=1
Address0.DefBcc=0
Address0.Tick=5
Address1=colleague@company.com
Address1.Fav=0
Address1.DefTo=0
Address1.DefCc=0
Address1.DefBcc=0
Address1.Tick=7
```
`MaxAddresses` only limits the **non**-favorites (oldest removed first);
favorites (`.Fav=1`) are kept without limit. `.DefTo`/`.DefCc`/`.DefBcc`
control the automatic pre-fill when a new compose window opens; `.Tick`
is an internal counter for "last used" (higher = more recent).

### Windows Credential Manager
Passwords and OAuth2 tokens are stored in the Windows Credential Manager.
Entry format: `MailFS/{account ID}/{password|oauth2_access|oauth2_refresh}`

Found under: **Control Panel → User Accounts → Credential Manager → Windows Credentials**

---

## Security

- Password authentication **only** over encrypted connections (TLS/STARTTLS)
- Certificate verification **strictly enabled** by default
- No plaintext storage of passwords or tokens
- Insecure connections (port 143/110 without encryption) must be explicitly enabled

---

## Building (developers)

Requirements:
- Lazarus 2.2+ / Free Pascal Compiler 3.2+ (https://www.lazarus-ide.org/)
  (built via `lazbuild`, not directly via `fpc` - MailFS depends on the
  LazUtils package (`Translations` unit for the `.po` language files),
  whose unit search paths only `lazbuild`/the IDE resolve automatically)
- OpenSSL 3.x headers / DLLs in the project directory

```bat
build.bat
```

Output: `build\mailfs.wfx`

Project structure:
```
mailfs/
├── mailfs.dpr              Main DLL project file
├── build.bat               Build script
├── src/
│   ├── wfx_api.pas         WFX SDK type definitions
│   ├── imap_types.pas      Data model (accounts, folders, messages)
│   ├── ssl_layer.pas       TCP+TLS socket layer (OpenSSL, loaded dynamically)
│   ├── imap_client.pas     IMAP4rev1 client
│   ├── pop3_client.pas     POP3 client
│   ├── smtp_client.pas     SMTP client (sending)
│   ├── credential_mgr.pas  Windows Credential Manager wrapper
│   ├── account_manager.pas Account management + INI persistence
│   ├── msg_cache.pas       Thread-safe metadata cache
│   ├── vfs_core.pas        Virtual filesystem + connection pool
│   ├── custom_columns.pas  TC Custom Columns (content plugin)
│   ├── dlg_accounts.pas    Dialog 1: account management
│   ├── dlg_edit_account.pas Dialog 2: edit/create account (incl. SMTP settings)
│   ├── dlg_compose.pas     Dialog 3: compose/send a message
│   └── mailfs_main.pas     WFX entry points (exports)
└── docs/
    └── README.md           This file
```

---

## Known limitations (MVP v1.0)

- **OAuth2:** A full browser-redirect implementation requires app registration with the respective provider (Google Cloud Console, Azure App Registration etc.). The architecture is prepared for it.
- **POP3:** Implemented at a basic level; folder hierarchy is not supported (POP3 has no concept of folders).
- **Writing:** No moving messages in v1. Deleting and composing/sending
  (see "Sending a message") are supported.
- **Composing (v1):** To/Cc/Bcc only as plain e-mail addresses, no
  "Name &lt;address&gt;" format; plain-text body only (no HTML); attachments are
  added one at a time via repeated **Attach...** clicks (no multi-select dialog).
- **Attachments (receiving):** Delivered as part of the complete EML file on download.
- **Very large mailboxes:** At most 500 messages per folder are loaded (newest first).

---

## Planned extensions

- Full OAuth2 flow with an embedded browser (v1.1)
- Mark as read (v1.2)
- "Name &lt;address&gt;" format and HTML body when composing (v1.1)
- Pagination for very large folders (v1.1)
- File logging for diagnostics (v1.1)

---

## License

MIT License – free to use, modify, and redistribute with attribution.
