✅ 完成項目 (Checklist)
任務	狀態	備註
2D 仿射變換矩陣：Translation / Rotation(Z) / Scale	✅	Matrix4.makeTrans / makeRotZ / makeScale
pnpoly（點在多邊形內測試）	✅	光線投射法
findBoundBox（包圍盒）	✅	請注意初始值選擇，見「注意事項」
Sutherland–Hodgman 裁剪	✅	以 isInside + computeIntersection 實作

📸 執行截圖 (Screenshots)

請插入 3–5 張關鍵畫面，建議包含：

產生 Rectangle / Star 並可於 Inspector 調整 Position / Rotation / Scale

物件被正確填色（pnpoly 生效）

裁剪後物件不會超出 Canvas（Sutherland–Hodgman 生效）


例如：
images/transform.png
images/fill.png
images/clipping.png

🔍 技術解說 (How the Code Produces Each Feature)
1) Matrix4：4×4 齊次座標矩陣（Affine Pipeline）

本作業採用 4×4 齊次座標矩陣以統一表示平移、旋轉、縮放等仿射變換。矩陣以一維陣列 m[16] 儲存，程式風格是 row-major 與行向量右乘（從 mult(Vector4) 與 MulPoint 的寫法可知），因此平移量位於最後一欄 m[3], m[7], m[11]。

單位矩陣：

makeIdentity(); // 對角線 1，其餘 0


平移矩陣（Translation）：

void makeTrans(Vector3 t) {
  makeIdentity();
  m[3] = t.x; // 第 1 列第 4 欄
  m[7] = t.y; // 第 2 列第 4 欄
  m[11]= t.z; // 第 3 列第 4 欄（2D 可不使用）
}


MulPoint 會把這些平移量加到座標上，讓 Inspector 的 Position 生效。

Z 軸旋轉（Rotation Z）：使 XY 平面內旋轉

void makeRotZ(float a) {
  makeIdentity();
  float c=(float)Math.cos(a), s=(float)Math.sin(a);
  m[0]=c;  m[1]=-s;
  m[4]=s;  m[5]= c;
  // 其餘維持 I
}


縮放（Scale）：

void makeScale(Vector3 s) {
  makeIdentity();
  m[0]=s.x;
  m[5]=s.y;
  m[10]=s.z; // 2D 可為 1
}


點與方向的套用差異：

MulPoint(p)：套用完整 4×4，含平移與 w 除法（透視除法預留）。

MulDirection(v)：只用左上 3×3 線性部分（旋轉/縮放/切變），不含平移。

補充：makeRotX / makeRotY 你可用以下安全實作（若日後需要 3D 支援）：

void makeRotX(float a) {
  makeIdentity();
  float c=(float)Math.cos(a), s=(float)Math.sin(a);
  m[5]=c;  m[6]=-s;
  m[9]=s;  m[10]=c;
}
void makeRotY(float a) {
  makeIdentity();
  float c=(float)Math.cos(a), s=(float)Math.sin(a);
  m[0]=c;  m[2]=s;
  m[8]=-s; m[10]=c;
}

2) CGLine：整數格點畫線（Bresenham 思想）
public void CGLine(float x1, float y1, float x2, float y2) {
  // 1) 端點取整，使像素落點穩定
  // 2) 以 dx, dy 判斷主軸；斜率 > 1 時交換主次軸
  // 3) 使用誤差項 d 控制何時推進副軸
  // 4) 逐點呼叫 drawPoint(x,y,color(0))
}


優點：只用整數加減，速度快、鋸齒均勻。此函式用於渲染線段與多邊形邊界。

3) pnpoly：光線投射法（Point-in-Polygon）
boolean pnpoly(float x, float y, Vector3[] V) {
  // 從點 (x,y) 向右射出水平光線
  // 逐邊檢查是否跨過該水平線，若跨過求交點 x_intersect
  // 若 x < x_intersect，翻轉 inside；最後 inside=true 代表在內部
}


關鍵兩段判斷：

(yi > y) != (yj > y)：此邊是否跨過水平 y

x < ((xj - xi)*(y - yi)/(yj - yi) + xi)：交點在測試點右側

此法適用凹/凸多邊形；搭配包圍盒可大幅加速。

4) findBoundBox：最小包圍盒（加速掃描）
public Vector3[] findBoundBox(Vector3[] v) {
  // 掃過所有頂點求 minX/minY/maxX/maxY
  // 回傳兩個對角點 (min), (max)
}


在填色或碰撞檢測時，只需要掃描 [minX..maxX] × [minY..maxY] 的像素區塊，將 pnpoly 呼叫數從整個畫布縮小到局部範圍，效能顯著提升。

5) Sutherland_Hodgman_algorithm：多邊形裁剪

流程：針對裁剪框 boundary 的每一條邊 A→B，將 subject 多邊形 points 進行一次「四情況」裁剪，逐輪輸出新的頂點串列。

四情況規則（S→E 表 subject 一條邊）：

S	E	輸出
in	in	E
in	out	intersection(S,E; A,B)
out	in	intersection(S,E; A,B), E
out	out	none



isInside(P,A,B)：以叉積符號決定 P 是否在「內側」半平面。

computeIntersection(S,E,A,B)：用兩線相交的解析式求交點。

裁剪完每一條 A→B 邊後，以輸出作為下輪輸入；走完所有邊便得到被 boundary 包住的裁切結果，使圖形不會畫出 Canvas 外。


⏩ 執行方式 (Run)

匯入或開啟專案，確保預設模板已成功編譯執行。

於 UI 介面點擊「Rectangle / Star」按鈕建立物件。

在 Hierarchy 選取物件，於 Inspector 調整 Position / Rotation / Scale。

驗證：

物件可平移／旋轉（Z）／縮放

物件被正確填色（pnpoly）

物件不跑出畫布（裁剪）

🤖 LLM 使用說明 (Disclosure)

本作業在以下階段使用 LLM（如 ChatGPT）：

釐清 Affine 矩陣與 row-major/column-major 的索引位置

了解 pnpoly（光線投射）的邏輯與 x_intersect 線性插值推導

Sutherland–Hodgman 四情況規則與 isInside 的繞序注意事項

撰寫此 README 的章節結構與文字潤飾

所有關鍵程式碼均由本人撰寫與整合，LLM 僅作為概念說明與文件撰寫輔助。

"# computer-graphic-HW-2" 
