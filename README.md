
"""
تاب الربط بالكود - اسم الملف: pairing_code.py
شغله بـ: python pairing_code.py
هيطلعلك كود 8 أرقام
"""
import os, subprocess, sys

NODE_SCRIPT = """
const { default: makeWASocket, useMultiFileAuthState } = require('@whiskeysockets/baileys')
const P = require('pino')
const readline = require('readline')
const rl = readline.createInterface({ input: process.stdin, output: process.stdout })
const question = (t) => new Promise(r => rl.question(t, r))
async function start(){
  const { state, saveCreds } = await useMultiFileAuthState('./auth_code_session')
  const sock = makeWASocket({ auth: state, logger: P({level:'silent'}), printQRInTerminal:false, browser:["Ubuntu","Chrome","20.0"] })
  sock.ev.on('creds.update', saveCreds)
  if(!sock.authState.creds.registered){
    let phone = await question('📱 رقمك بالكود الدولي (مثال 201002707721): ')
    phone=phone.trim().replace(/\D/g,'')
    let code = await sock.requestPairingCode(phone)
    code = code?.match(/.{1,4}/g)?.join('-') || code
    console.log(`\n✅ METEORMD: ${code}`)
    console.log('واتساب > الأجهزة المرتبطة > ربط برقم هاتف > اكتب الكود')
  }
  sock.ev.on('connection.update', ({connection})=>{
    if(connection==='open'){ console.log('✅ اتربط بنجاح!'); rl.close() }
  })
}
start()
"""

with open("temp_pair.js","w",encoding="utf-8") as f:
    f.write(NODE_SCRIPT)

if not os.path.exists("node_modules"):
    print("📦 بنزل المكتبات...")
    os.system("npm install @whiskeysockets/baileys pino")

os.system("node temp_pair.js")
