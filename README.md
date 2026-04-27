<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <title>Chess Beta</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
    <script src="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/@chrisoakman/chessboardjs@1.0.0/dist/chessboard-1.0.0.min.css">
    <style>
        body { background: #312e2b; color: white; font-family: sans-serif; text-align: center; }
        #board { width: 400px; margin: 20px auto; }
        .info { background: #262421; padding: 15px; border-radius: 8px; display: inline-block; }
        button { background: #81b64c; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer; font-weight: bold; }
    </style>
</head>
<body>
    <h2>Chess Beta Test</h2>
    <div id="board"></div>
    <div class="info">
        <p id="status">Hamle bekleniyor...</p>
        <button onclick="location.reload()">Sıfırla</button>
    </div>
    <script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
    <script>
        var board = null;
        var game = new Chess();
        function onDrop(source, target) {
            var move = game.move({ from: source, to: target, promotion: 'q' });
            if (move === null) return 'snapback';
            
            fetch('/api/move', {
                method: 'POST',
                body: JSON.stringify({ fen: game.fen(), move: source + target })
            }).then(res => res.json()).then(data => {
                game.load(data.fen);
                board.position(data.fen);
            });
        }
        board = Chessboard('board', { draggable: true, position: 'start', onDrop: onDrop });
    </script>
</body>
</html>
{
  "name": "chess-beta",
  "version": "1.0.0",
  "dependencies": {
    "chess.js": "^0.13.4"
  }
}
from http.server import BaseHTTPRequestHandler
import json
import random
# Vercel'de python-chess kütüphanesi hazır olmayabilir, 
# o yüzden basit bir mantık kuruyoruz.
class handler(BaseHTTPRequestHandler):
    def do_POST(self):
        self.send_response(200)
        self.send_header('Content-type', 'application/json')
        self.end_headers()
        # Şimdilik sadece gelen FEN'i geri gönderen bir taslak
        content_length = int(self.headers['Content-Length'])
        post_data = json.loads(self.rfile.read(content_length))
        
        # Beta olduğu için hamleyi sadece onaylayıp geri döner
        response = {"fen": post_data.get("fen"), "status": "ok"}
        self.wfile.write(json.dumps(response).encode()
