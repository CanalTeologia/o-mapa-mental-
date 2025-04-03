from pathlib import Path

# Código HTML completo e funcional com o botão "+ Novo Bloco" funcionando e todas as funcionalidades atualizadas
index_html = """<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Mapa Mental - Edição Visual</title>
  <script src="https://cdn.jsdelivr.net/npm/drawflow/dist/drawflow.min.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/drawflow/dist/drawflow.min.css">
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background-color: #1e1e1e;
      font-family: sans-serif;
    }

    #drawflow {
      width: 100vw;
      height: 100vh;
    }

    .drawflow-node {
      background: transparent !important;
      border: 1px solid #666;
      border-radius: 6px;
      padding: 4px;
      color: white;
      max-width: 200px;
      cursor: pointer;
    }

    .drawflow-node .editable-text {
      outline: none;
      white-space: pre;
      overflow-x: auto;
      display: inline-block;
      min-width: 50px;
      max-width: 100%;
      color: white !important;
    }

    .node-buttons {
      text-align: right;
      margin-top: 4px;
    }

    .node-buttons button {
      background: #444;
      color: white;
      border: none;
      font-size: 10px;
      padding: 2px 5px;
      border-radius: 4px;
      cursor: pointer;
    }

    .node-buttons button:hover {
      background: #666;
    }

    .toolbar {
      position: absolute;
      top: 10px;
      left: 10px;
      z-index: 10;
      background: #2b2b2b;
      padding: 10px;
      border-radius: 6px;
      color: white;
    }

    .toolbar button {
      background: #444;
      color: white;
      border: none;
      padding: 5px 8px;
      border-radius: 3px;
      font-size: 13px;
    }

    #editorModal {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: #333;
      color: white;
      padding: 20px;
      border-radius: 8px;
      z-index: 999;
      display: none;
    }

    #editorModal input[type="color"] {
      margin-bottom: 10px;
    }

    #editorModal textarea {
      width: 100%;
      height: 100px;
      background: #222;
      color: white;
      border: 1px solid #555;
      border-radius: 4px;
      padding: 5px;
    }

    #editorModal button {
      margin-top: 10px;
      padding: 5px 10px;
      background: #555;
      border: none;
      border-radius: 4px;
      color: white;
    }
  </style>
</head>
<body>

<div id="drawflow"></div>

<div class="toolbar">
  <button onclick="openEditor()">+ Novo Bloco</button>
</div>

<div id="editorModal">
  <div><strong>Texto:</strong></div>
  <textarea id="nodeText"></textarea>

  <div style="margin-top:10px;">
    <label>Borda: <input type="color" id="borderColor" value="#666666"></label><br>
    <label>Fundo: <input type="color" id="bgColor" value="#1e1e1e"></label><br>
    <label>Texto: <input type="color" id="textColor" value="#ffffff"></label>
  </div>

  <button onclick="createOrUpdateNode()">Salvar</button>
</div>

<script>
  const editor = new Drawflow(document.getElementById("drawflow"));
  editor.reroute = true;
  editor.zoom_enable = true;
  editor.start();

  let editingNodeId = null;

  function openEditor(data = null, nodeId = null) {
    editingNodeId = nodeId;
    document.getElementById('editorModal').style.display = "block";

    if (data) {
      document.getElementById("nodeText").value = data.content;
      document.getElementById("borderColor").value = data.borderColor;
      document.getElementById("bgColor").value = data.bgColor;
      document.getElementById("textColor").value = data.textColor;
    } else {
      document.getElementById("nodeText").value = "";
      document.getElementById("borderColor").value = "#666666";
      document.getElementById("bgColor").value = "#1e1e1e";
      document.getElementById("textColor").value = "#ffffff";
    }
  }

  function createOrUpdateNode() {
    const content = document.getElementById("nodeText").value;
    const borderColor = document.getElementById("borderColor").value;
    const bgColor = document.getElementById("bgColor").value;
    const textColor = document.getElementById("textColor").value;

    const html = \`
      <div style="border: 1px solid \${borderColor}; background: \${bgColor}; color: \${textColor}; border-radius:5px; padding:5px;">
        <div class="editable-text" contenteditable="true" onblur="updateNodeContent(\${editingNodeId})">\${content}</div>
        <div class="node-buttons">
          <button onclick="toggleChildren(\${editingNodeId})">🔽</button>
        </div>
      </div>\`;

    const data = { content, borderColor, bgColor, textColor };

    if (editingNodeId === null) {
      const newId = editor.addNode("text", 1, 1, 250, 150, "text", data, html);
    } else {
      editor.updateNodeDataFromId(editingNodeId, data);
      editor.updateNodeFromId(editingNodeId, html);
    }

    document.getElementById("editorModal").style.display = "none";
    editingNodeId = null;
  }

  function updateNodeContent(id) {
    const node = editor.getNodeFromId(id);
    const el = document.querySelector(\`.drawflow-node[node_id="\${id}"] .editable-text\`);
    if (el) {
      node.data.content = el.innerText;
    }
  }

  function toggleChildren(parentId) {
    const children = editor.getNodesFrom(parentId);
    children.forEach(childId => {
      const nodeEl = document.querySelector(\`.drawflow-node[node_id="\${childId}"]\`);
      const connOut = document.querySelectorAll(\`.connection.node_out_\${parentId}\`);
      const connIn = document.querySelectorAll(\`.connection.node_in_\${childId}\`);

      const isHidden = nodeEl.style.display === "none";

      if (nodeEl) nodeEl.style.display = isHidden ? "block" : "none";
      connOut.forEach(conn => conn.style.display = isHidden ? "block" : "none");
      connIn.forEach(conn => conn.style.display = isHidden ? "block" : "none");
    });
  }

  document.addEventListener("dblclick", function (e) {
    const nodeEl = e.target.closest(".drawflow-node");
    if (!nodeEl) return;
    const nodeId = parseInt(nodeEl.getAttribute("node_id"));
    const node = editor.getNodeFromId(nodeId);
    openEditor(node.data, nodeId);
  });

  document.getElementById("drawflow").addEventListener("wheel", function (e) {
    e.preventDefault();
    e.deltaY < 0 ? editor.zoom_in() : editor.zoom_out();
  });
</script>

</body>
</html>
"""

# Criar o arquivo index.html com todo o conteúdo
output_file = Path("/mnt/data/index.html")
output_file.write_text(index_html, encoding="utf-8")

output_file
