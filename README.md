import express from "express";
import fs from "fs";
import path from "path";
import axios from "axios";

const app = express();
app.use(express.json());

const __dirname = path.resolve();
const config = JSON.parse(fs.readFileSync("./config.json", "utf8"));
const prefix = config.prefix;

const commands = new Map();

// Load all commands
const files = fs.readdirSync("./commands").filter(f => f.endsWith(".js"));
for (let file of files) {
  const cmd = await import(`./commands/${file}`);
  commands.set(cmd.name, cmd);
}

app.post("/bot", async (req, res) => {
  const message = req.body.message;
  if (!message.startsWith(prefix)) return res.json({ reply: null });

  const args = message.slice(prefix.length).trim().split(/ +/);
  const cmdName = args.shift().toLowerCase();

  const cmd = commands.get(cmdName);
  if (!cmd) return res.json({ reply: "Unknown command." });

  try {
    const result = await cmd.run(args);
    return res.json({ reply: result });
  } catch (err) {
    return res.json({ reply: "Error executing command." });
  }
});

// Ping check
app.get("/", (req, res) => {
  res.send("RHMIM MLS Bot is running!");
});

app.listen(3000, () => {
  console.log("Bot Server Running on Port 3000");
});
