# fehurihi-lodge-backend
const express = require("express");
const cors = require("cors");
const fs = require("fs");
const app = express();
const PORT = 5000;

app.use(cors());
app.use(express.json());

const roomsFile = "rooms.json";

const getRooms = () => {
  return JSON.parse(fs.readFileSync(roomsFile, "utf-8"));
};

const saveRooms = (rooms) => {
  fs.writeFileSync(roomsFile, JSON.stringify(rooms, null, 2));
};

// Get all rooms
app.get("/api/rooms", (req, res) => {
  res.json(getRooms());
});

// Book a room
app.post("/api/book", (req, res) => {
  const { name, phone, roomId } = req.body;
  if (!name || !phone || !roomId) return res.status(400).json({ error: "Missing details" });

  let rooms = getRooms();
  const roomIndex = rooms.findIndex((room) => room.id === roomId);
  if (roomIndex === -1 || !rooms[roomIndex].available) {
    return res.status(400).json({ error: "Room not available" });
  }

  rooms[roomIndex].available = false;
  rooms[roomIndex].availableDate = new Date(Date.now() + 2 * 24 * 60 * 60 * 1000).toISOString().split("T")[0]; // 2-day block
  saveRooms(rooms);

  res.json({ message: "Room booked successfully", room: rooms[roomIndex] });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});
