/*
====================================================================
  AI KUNG FU / WUSHU TRAINING + 3D BIOMECHANICS ANALYZER
  OOP REFACTORED VERSION
  APDE / PROCESSING ANDROID / JAVA / P3D SCREEN ARCHITECTURE
====================================================================
*/

// ================================================================
// APPLICATION STATE & GLOBALS
// ================================================================
final int HOME_SCREEN = 0;
final int CATEGORY_SCREEN = 1;
final int TECHNIQUE_SCREEN = 2;

int screen = HOME_SCREEN;
int selectedCategory = -1;
int selectedTechnique = -1;

float rotX = -0.12;
float rotY = 0.0;
final float defaultRotX = -0.12;
final float defaultRotY = 0.0;
boolean draggingModel = false;
float animationClock = 0.0;

// Core Objects
Skeleton skeleton;
ArrayList<Button> homeButtons;
ArrayList<Button> categoryButtons;

// Small UI Buttons
Button btnBackToHome, btnBackToCat;
Button btnPose, btnWalk, btnPrev, btnNext, btnReset;

// ================================================================
// SETUP
// ================================================================
void setup() {
  orientation(PORTRAIT);
  fullScreen(P3D);
  smooth(8);

  // Initialize Data & Objects
  skeleton = new Skeleton(DataStore.connections);
  initUI();
}

// ================================================================
// DRAW LOOP
// ================================================================
void draw() {
  background(12, 16, 22);
  animationClock += 0.016;

  switch (screen) {
    case HOME_SCREEN:
      drawHomeScreen();
      break;
    case CATEGORY_SCREEN:
      drawCategoryScreen();
      break;
    case TECHNIQUE_SCREEN:
      skeleton.update();
      drawTechniqueScreen();
      break;
  }
}

// ================================================================
// SCREENS
// ================================================================
void drawHomeScreen() {
  hint(DISABLE_DEPTH_TEST);
  
  fill(255, 120, 50);
  textAlign(CENTER, CENTER);
  textSize(width * 0.075);
  text("AI KUNG FU / WUSHU", width / 2.0, height * 0.065);

  fill(255);
  textSize(width * 0.035);
  text("300+ MOVEMENT TAXONOMY & BIOMECHANICS", width / 2.0, height * 0.115);

  for (Button btn : homeButtons) {
    btn.draw(btn.id == selectedCategory);
  }

  fill(160, 175, 190);
  textSize(width * 0.033);
  text("Animal Archetypes • Taolu • Kinematics", width / 2.0, height * 0.90);
  text("Select a classification model", width / 2.0, height * 0.95);
  
  hint(ENABLE_DEPTH_TEST);
}

void drawCategoryScreen() {
  hint(DISABLE_DEPTH_TEST);
  
  fill(255, 140, 60);
  textAlign(CENTER, CENTER);
  textSize(width * 0.060);
  text(DataStore.categoryNames[selectedCategory], width / 2.0, height * 0.055);

  btnBackToHome.draw(false);

  for (Button btn : categoryButtons) {
    btn.draw(false);
  }
  
  hint(ENABLE_DEPTH_TEST);
}

void drawTechniqueScreen() {
  // 3D Render
  skeleton.render(rotX, rotY);

  // HUD Render
  hint(DISABLE_DEPTH_TEST);
  
  btnBackToCat.draw(false);

  fill(255, 140, 60);
  textAlign(LEFT, TOP);
  textSize(width * 0.052);
  text(DataStore.techniques[selectedCategory][selectedTechnique], width * 0.54, height * 0.025);

  fill(220);
  textSize(width * 0.030);
  text(DataStore.categoryNames[selectedCategory], width * 0.54, height * 0.085);

  fill(255, 200, 70);
  textSize(width * 0.033);
  text("AI MEASUREMENT", width * 0.54, height * 0.14);

  fill(240);
  textSize(width * 0.029);
  text(DataStore.trainingType[selectedCategory][selectedTechnique], width * 0.54, height * 0.185, width * 0.41, height * 0.15);

  fill(255, 200, 70);
  textSize(width * 0.033);
  text("KINEMATICS", width * 0.54, height * 0.30);
  
  fill(235);
  textSize(width * 0.027);
  text("• COM trajectory", width * 0.54, height * 0.35);
  text("• Joint curvature (κ)", width * 0.54, height * 0.39);
  text("• Base of support", width * 0.54, height * 0.43);

  // Playback & Navigation
  btnPose.draw(false);
  btnWalk.draw(false);
  btnPrev.draw(false);
  btnNext.draw(false);
  btnReset.draw(false);

  fill(150, 170, 185);
  textSize(width * 0.027);
  text("Drag skeleton to rotate 3D view", width * 0.54, height * 0.80);

  fill(255, 140, 60);
  text(skeleton.walking ? "LOCOMOTION ACTIVE" : "TECHNIQUE RENDERED", width * 0.54, height * 0.85);

  hint(ENABLE_DEPTH_TEST);
}

// ================================================================
// CONTROLLER: INTERACTION
// ================================================================
void mousePressed() {
  if (screen == HOME_SCREEN) {
    for (Button btn : homeButtons) {
      if (btn.isHovered(mouseX, mouseY)) {
        selectedCategory = btn.id;
        buildCategoryButtons();
        screen = CATEGORY_SCREEN;
        return;
      }
    }
  } 
  else if (screen == CATEGORY_SCREEN) {
    if (btnBackToHome.isHovered(mouseX, mouseY)) {
      screen = HOME_SCREEN;
      return;
    }
    for (Button btn : categoryButtons) {
      if (btn.isHovered(mouseX, mouseY)) {
        selectedTechnique = btn.id;
        skeleton.startTechnique(selectedCategory, selectedTechnique);
        screen = TECHNIQUE_SCREEN;
        return;
      }
    }
  } 
  else if (screen == TECHNIQUE_SCREEN) {
    if (btnBackToCat.isHovered(mouseX, mouseY)) {
      skeleton.walking = false;
      screen = CATEGORY_SCREEN;
      return;
    }
    
    if (btnPose.isHovered(mouseX, mouseY)) { skeleton.startTechnique(selectedCategory, selectedTechnique); return; }
    if (btnWalk.isHovered(mouseX, mouseY)) { skeleton.startWalking(); return; }
    
    if (btnPrev.isHovered(mouseX, mouseY)) { 
      selectedTechnique = (selectedTechnique - 1 + 10) % 10; 
      skeleton.startTechnique(selectedCategory, selectedTechnique); 
      return; 
    }
    if (btnNext.isHovered(mouseX, mouseY)) { 
      selectedTechnique = (selectedTechnique + 1) % 10; 
      skeleton.startTechnique(selectedCategory, selectedTechnique); 
      return; 
    }

    if (btnReset.isHovered(mouseX, mouseY)) { 
      rotX = defaultRotX; 
      rotY = defaultRotY; 
      return; 
    }

    if (mouseY > height * 0.10 && mouseX < width * 0.52) {
      draggingModel = true;
    }
  }
}

void mouseReleased() { 
  draggingModel = false; 
}

void mouseDragged() {
  if (screen == TECHNIQUE_SCREEN && draggingModel) {
    rotY += (mouseX - pmouseX) * 0.01; 
    rotX -= (mouseY - pmouseY) * 0.01;
    rotX = constrain(rotX, -1.45, 1.45);
  }
}

// ================================================================
// UI INITIALIZATION
// ================================================================
void initUI() {
  homeButtons = new ArrayList<Button>();
  categoryButtons = new ArrayList<Button>();

  // Home Screen Menu Buttons
  float margin = width * 0.055;
  float gap = width * 0.025;
  float bw = (width - 2 * margin - gap) / 2.0;
  float bh = height * 0.075;

  for (int i = 0; i < DataStore.categoryNames.length; i++) {
    int col = i % 2; 
    int row = i / 2;
    float x = margin + col * (bw + gap);
    if (i == DataStore.categoryNames.length - 1 && DataStore.categoryNames.length % 2 != 0) {
        x = width / 2.0 - bw / 2.0;
    }
    float y = height * 0.16 + row * (bh + gap); 
    homeButtons.add(new Button(i, x, y, bw, bh, DataStore.categoryNames[i], true));
  }

  // Persistent Small Buttons
  btnBackToHome = new Button(-1, 10, 10, width * 0.20, height * 0.055, "HOME", false);
  btnBackToCat = new Button(-1, 10, 10, width * 0.18, height * 0.055, "BACK", false);
  
  btnPose = new Button(0, width * 0.54, height * 0.52, width * 0.19, height * 0.055, "POSE", false);
  btnWalk = new Button(1, width * 0.75, height * 0.52, width * 0.19, height * 0.055, "WALK", false);
  btnPrev = new Button(2, width * 0.54, height * 0.60, width * 0.19, height * 0.055, "PREV", false);
  btnNext = new Button(3, width * 0.75, height * 0.60, width * 0.19, height * 0.055, "NEXT", false);
  btnReset = new Button(4, width * 0.54, height * 0.68, width * 0.40, height * 0.055, "RESET 3D VIEW", false);
}

void buildCategoryButtons() {
  categoryButtons.clear();
  float margin = width * 0.05;
  float gap = width * 0.025;
  float bw = (width - 2 * margin - gap) / 2.0;
  float bh = height * 0.073;

  for (int i = 0; i < 10; i++) {
    int col = i % 2; 
    int row = i / 2; 
    float x = margin + col * (bw + gap); 
    float y = height * 0.12 + row * (bh + gap); 
    categoryButtons.add(new Button(i, x, y, bw, bh, DataStore.techniques[selectedCategory][i], true));
  }
}


// ================================================================
// OBJECT: BUTTON
// ================================================================
class Button {
  int id;
  float x, y, w, h;
  String label;
  boolean isMenuType;

  Button(int id, float x, float y, float w, float h, String label, boolean isMenuType) {
    this.id = id;
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.label = label;
    this.isMenuType = isMenuType;
  }

  void draw(boolean isActive) {
    if (isMenuType) {
      fill(isActive ? color(160, 60, 20) : color(35, 43, 53));
      stroke(255, 120, 50);
      strokeWeight(2);
      rect(x, y, w, h, 12);
      fill(255);
      textAlign(CENTER, CENTER);
      textSize(min(width * 0.033, h * 0.35));
      text(label, x + w / 2.0, y + h / 2.0);
    } else {
      fill(38, 48, 60);
      stroke(255, 120, 50);
      strokeWeight(2);
      rect(x, y, w, h, 10);
      fill(255);
      textAlign(CENTER, CENTER);
      textSize(width * 0.027);
      text(label, x + w / 2.0, y + h / 2.0);
    }
  }

  boolean isHovered(float mx, float my) {
    return (mx >= x && mx <= x + w && my >= y && my <= y + h);
  }
}

// ================================================================
// OBJECT: SKELETON (Kinematics & Pose Logic)
// ================================================================
class Skeleton {
  final int NUM_POINTS = 33;
  PVector[] current, target, start, finalTarget, neutral;
  float[] restLength;
  int[][] connections;

  float transition = 1.0;
  boolean walking = false;
  float walkPhase = 0.0;

  Skeleton(int[][] conns) {
    connections = conns;
    current = new PVector[NUM_POINTS];
    target = new PVector[NUM_POINTS];
    start = new PVector[NUM_POINTS];
    finalTarget = new PVector[NUM_POINTS];
    neutral = new PVector[NUM_POINTS];

    for (int i = 0; i < NUM_POINTS; i++) {
      current[i] = new PVector(); target[i] = new PVector();
      start[i] = new PVector(); finalTarget[i] = new PVector();
      neutral[i] = new PVector();
    }

    buildNeutralPose(neutral);

    copyPose(neutral, current);
    copyPose(neutral, target);
    copyPose(neutral, start);
    copyPose(neutral, finalTarget);

    restLength = new float[connections.length];
    for (int i = 0; i < connections.length; i++) {
      int a = connections[i][0]; int b = connections[i][1];
      restLength[i] = PVector.dist(neutral[a], neutral[b]);
    }

    buildWushuGuard(finalTarget);
    copyPose(finalTarget, target);
    copyPose(finalTarget, current);
    copyPose(finalTarget, start);
  }

  void update() {
    if (walking) {
      walkPhase += 0.075;
      buildWalkPose(walkPhase, finalTarget);
    }
    
    transition += 0.055;
    transition = constrain(transition, 0, 1);
    
    for (int i = 0; i < NUM_POINTS; i++) {
      target[i].set(start[i]);
      target[i].lerp(finalTarget[i], transition);
    }
    
    solveConstraints();
  }

  void solveConstraints() {
    for (int i = 0; i < NUM_POINTS; i++) current[i].set(target[i]); 
    
    final int ITERATIONS = 18;
    for (int iteration = 0; iteration < ITERATIONS; iteration++) {
      for (int e = 0; e < connections.length; e++) { 
        int ia = connections[e][0]; int ib = connections[e][1]; 
        PVector a = current[ia]; PVector b = current[ib]; 
        
        float dx = b.x - a.x; float dy = b.y - a.y; float dz = b.z - a.z; 
        float d = sqrt(dx * dx + dy * dy + dz * dz); 
        if (d < 0.0001) d = 0.0001; 
        
        float error = (d - restLength[e]) / d; 
        float stiffness = 0.90; 
        float cx = dx * error * 0.5 * stiffness; 
        float cy = dy * error * 0.5 * stiffness; 
        float cz = dz * error * 0.5 * stiffness; 
        
        boolean anchorA = (ia == 23 || ia == 24); // Hips anchor
        boolean anchorB = (ib == 23 || ib == 24);
        
        if (!anchorA && !anchorB) { a.x+=cx; a.y+=cy; a.z+=cz; b.x-=cx; b.y-=cy; b.z-=cz; } 
        else if (anchorA && !anchorB) { b.x-=cx*2; b.y-=cy*2; b.z-=cz*2; } 
        else if (!anchorA && anchorB) { a.x+=cx*2; a.y+=cy*2; a.z+=cz*2; } 
      }
      current[23].set(target[23]); 
      current[24].set(target[24]); 
    }
  }

  void render(float rx, float ry) {
    pushMatrix();
    translate(width * 0.38, height * 0.57, -500);
    rotateX(rx);
    rotateY(ry);
    
    // Draw Floor
    float floorY = height / 12.0 * 6.35; 
    stroke(45, 60, 70); 
    strokeWeight(2);
    for (float i = -1200; i <= 1200; i += 120) { 
      line(i, floorY, -1200, i, floorY, 1200); 
      line(-1200, floorY, i, 1200, floorY, i); 
    }

    // Draw Bones
    stroke(255, 140, 60); 
    strokeWeight(8);
    for (int e = 0; e < connections.length; e++) {
      PVector p1 = current[connections[e][0]]; PVector p2 = current[connections[e][1]];
      line(p1.x, p1.y, p1.z, p2.x, p2.y, p2.z);
    }

    // Draw Joints
    for (int i = 0; i < NUM_POINTS; i++) {
      pushMatrix(); 
      translate(current[i].x, current[i].y, current[i].z); noStroke();
      boolean major = (i >= 11 && i <= 16) || (i >= 23 && i <= 28);
      fill(major ? color(255, 80, 50) : color(255, 200, 80)); 
      sphere(major ? 14 : 9);
      
      hint(DISABLE_DEPTH_TEST); 
      fill(255); textAlign(CENTER, CENTER); textSize(18); 
      text("" + i, 0, -22); 
      hint(ENABLE_DEPTH_TEST);
      popMatrix();
    }
    popMatrix();
  }

  void startTechnique(int cat, int tech) { 
    copyPose(current, start); 
    buildSelectedTechnique(cat, tech, finalTarget); 
    transition = 0.0; 
    walking = false; 
  }
  
  void startWalking() { 
    copyPose(current, start); 
    walking = true; 
    walkPhase = 0; 
    transition = 0.0; 
  }

  void copyPose(PVector[] source, PVector[] dest) {
    for (int i = 0; i < NUM_POINTS; i++) dest[i].set(source[i]);
  }

  // --- POSTURAL BUILDERS ---
  void buildSelectedTechnique(int c, int t, PVector[] p) {
    if (c == 0) buildTigerPose(p);
    else if (c == 1) buildCranePose(p);
    else if (c == 2) buildSnakePose(p);
    else if (c == 3) buildMonkeyPose(p);
    else if (c == 4) buildEaglePose(p);
    else if (c == 5) buildDragonPose(p);
    else if (c == 7) buildDrunkenPose(p);
    else if (c == 8) buildFrogPose(p);
    else if (c == 6) {
      if (t == 0) buildGongbuPose(p);
      else if (t == 1) buildMabuPose(p);
      else if (t == 2) buildXubuPose(p);
      else if (t == 3) buildPubuPose(p);
      else if (t == 4) buildDulibuPose(p);
      else buildWushuGuard(p); 
    }
  }

  void buildNeutralPose(PVector[] p) {
    float s = height / 12.0;
    p[0].set(0, -3.45*s, 0); p[1].set(-0.2*s, -3.25*s, 0); p[2].set(-0.32*s, -3.24*s, 0);
    p[3].set(-0.45*s, -3.18*s, 0); p[4].set(0.2*s, -3.25*s, 0); p[5].set(0.32*s, -3.24*s, 0);
    p[6].set(0.45*s, -3.18*s, 0); p[7].set(-0.62*s, -3.05*s, 0); p[8].set(0.62*s, -3.05*s, 0);
    p[9].set(-0.2*s, -2.72*s, 0); p[10].set(0.2*s, -2.72*s, 0);
    p[11].set(-1.2*s, -1.5*s, 0); p[12].set(1.2*s, -1.5*s, 0);
    p[23].set(-0.65*s, 1.55*s, 0); p[24].set(0.65*s, 1.55*s, 0); 
    p[13].set(-1.55*s, 0, 0); p[14].set(1.55*s, 0, 0); 
    p[15].set(-1.7*s, 1.45*s, 0); p[16].set(1.7*s, 1.45*s, 0); 
    p[25].set(-0.65*s, 3.55*s, 0); p[26].set(0.65*s, 3.55*s, 0); 
    p[27].set(-0.65*s, 5.55*s, 0); p[28].set(0.65*s, 5.55*s, 0); 
    syncFingers(p); syncFeet(p);
  }

  void buildGongbuPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].y += 0.65*s; p[24].y += 0.65*s;
    p[25].set(-0.6*s, 3.85*s, 1.7*s); p[27].set(-0.6*s, 5.5*s, 2.65*s);
    p[26].set(0.6*s, 3.2*s, -1.4*s); p[28].set(0.6*s, 5.5*s, -2.75*s);
    syncFingers(p); syncFeet(p);
  }

  void buildMabuPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].y += 1.1*s; p[24].y += 1.1*s;
    p[25].set(-1.9*s, 4.2*s, 0); p[27].set(-2.4*s, 5.5*s, 0);
    p[26].set(1.9*s, 4.2*s, 0); p[28].set(2.4*s, 5.5*s, 0);
    p[13].set(-0.9*s, -0.8*s, 0.8*s); p[15].set(0, -0.75*s, 2.2*s);
    p[14].set(1.3*s, -0.2*s, -0.55*s); p[16].set(1.0*s, 0.85*s, -0.15*s); 
    syncFingers(p); syncFeet(p);
  }

  void buildXubuPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].z = -0.65*s; p[24].z = -0.65*s; p[23].y += 0.35*s; p[24].y += 0.35*s;
    p[25].set(-0.55*s, 3.8*s, 1.15*s); p[27].set(-0.55*s, 5.35*s, 2.25*s); 
    p[26].set(1.2*s, 4.1*s, -1.15*s); p[28].set(1.2*s, 5.35*s, -2.0*s); 
    syncFingers(p); syncFeet(p);
  }

  void buildPubuPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].y += 2.2*s; p[24].y += 2.2*s;
    p[25].set(-3.0*s, 4.0*s, 0); p[27].set(-3.5*s, 5.5*s, 0); 
    p[26].set(0.65*s, 4.5*s, 0); p[28].set(0.65*s, 5.5*s, 0); 
    syncFingers(p); syncFeet(p);
  }

  void buildDulibuPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[25].set(-0.65*s, 2.5*s, 1.5*s); p[27].set(-0.65*s, 4.0*s, 1.0*s); 
    p[26].set(0.65*s, 3.55*s, 0); p[28].set(0.65*s, 5.55*s, 0); 
    syncFingers(p); syncFeet(p);
  }

  void buildWushuGuard(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[25].set(-0.78*s, 3.45*s, 0.95*s); p[27].set(-0.78*s, 5.35*s, 1.25*s);
    p[26].set(0.78*s, 3.35*s, -0.85*s); p[28].set(0.78*s, 5.35*s, -1.3*s);
    p[13].set(-1.2*s, -0.05*s, 0.65*s); p[15].set(-0.48*s, -1.25*s, 1.2*s);
    p[14].set(1.05*s, 0.05*s, 0.35*s); p[16].set(0.35*s, -0.5*s, 0.75*s);
    syncFingers(p); syncFeet(p);
  }

  // --- ANIMAL ARCHETYPES ---
  void buildTigerPose(PVector[] p) {
    buildGongbuPose(p); float s = height / 12.0;
    p[11].z += 0.5*s; p[12].z -= 0.2*s; 
    p[13].set(-1.0*s, 0.5*s, 1.5*s); p[15].set(-0.5*s, 0.5*s, 2.5*s);
    p[14].set(1.5*s, -0.5*s, 0.5*s); p[16].set(1.0*s, -0.5*s, 1.5*s);
    syncFingers(p); syncFeet(p);
  }

  void buildCranePose(PVector[] p) {
    buildDulibuPose(p); float s = height / 12.0;
    p[13].set(-2.5*s, -1.0*s, 0.5*s); p[15].set(-3.5*s, -0.2*s, 0.8*s);
    p[14].set(2.5*s, -1.0*s, -0.5*s); p[16].set(3.5*s, -0.2*s, -0.8*s);
    syncFingers(p); syncFeet(p);
  }

  void buildSnakePose(PVector[] p) {
    buildPubuPose(p); float s = height / 12.0;
    p[13].set(-2.0*s, 0.5*s, 0.5*s); p[15].set(-3.0*s, 0.5*s, 1.0*s);
    p[14].set(1.0*s, -1.5*s, 0); p[16].set(0, -2.5*s, 0.5*s); 
    syncFingers(p); syncFeet(p);
  }

  void buildMonkeyPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].y += 1.5*s; p[24].y += 1.5*s; 
    p[11].y += 0.5*s; p[12].y += 0.5*s; 
    p[25].set(-0.5*s, 4.0*s, 1.0*s); p[27].set(-0.3*s, 5.5*s, 1.5*s);
    p[26].set(0.5*s, 4.0*s, 0.5*s); p[28].set(0.3*s, 5.5*s, 1.0*s);
    p[13].set(-0.8*s, 0.5*s, 0.8*s); p[15].set(-0.5*s, 0.2*s, 1.5*s);
    p[14].set(0.8*s, 0.5*s, 0.8*s); p[16].set(0.5*s, 0.2*s, 1.5*s);
    syncFingers(p); syncFeet(p);
  }

  void buildEaglePose(PVector[] p) {
    buildXubuPose(p); float s = height / 12.0;
    p[13].set(-1.0*s, -0.5*s, 2.0*s); p[15].set(-0.5*s, -0.5*s, 3.5*s); 
    p[14].set(1.5*s, -1.0*s, -1.0*s); p[16].set(2.5*s, -1.5*s, -1.5*s); 
    syncFingers(p); syncFeet(p);
  }

  void buildDragonPose(PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    p[23].y += 1.0*s; p[24].y += 1.0*s; p[23].x += 0.5*s; p[24].x += 0.5*s;
    p[25].set(-0.5*s, 3.5*s, 1.5*s); p[27].set(0.5*s, 5.5*s, 1.0*s);
    p[26].set(1.0*s, 3.8*s, -1.0*s); p[28].set(0.5*s, 5.5*s, -2.0*s);
    p[11].y -= 0.5*s; p[13].set(-1.0*s, -2.0*s, 1.0*s); p[15].set(-0.5*s, -3.0*s, 1.5*s);
    p[14].set(1.5*s, 1.0*s, 0.5*s); p[16].set(1.0*s, 2.0*s, 1.0*s);
    syncFingers(p); syncFeet(p);
  }

  void buildDrunkenPose(PVector[] p) {
    buildXubuPose(p); float s = height / 12.0;
    p[23].x -= 0.6*s; p[24].x -= 0.6*s; p[11].x -= 1.2*s; p[12].x -= 1.2*s;
    p[11].z -= 0.5*s; p[12].z -= 0.5*s; 
    p[13].set(-1.5*s, -2.5*s, 0.5*s); p[15].set(-0.5*s, -3.5*s, 1.5*s); 
    p[14].set(1.5*s, 0.5*s, -0.5*s); p[16].set(2.0*s, 1.5*s, -1.0*s);
    syncFingers(p); syncFeet(p);
  }

  void buildFrogPose(PVector[] p) {
    buildMabuPose(p); float s = height / 12.0;
    p[23].y += 1.8*s; p[24].y += 1.8*s; 
    p[13].set(-1.2*s, 2.0*s, 1.2*s); p[15].set(-1.5*s, 4.0*s, 1.5*s);
    p[14].set(1.2*s, 2.0*s, 1.2*s); p[16].set(1.5*s, 4.0*s, 1.5*s);
    syncFingers(p); syncFeet(p);
  }

  void buildWalkPose(float phase, PVector[] p) {
    buildNeutralPose(p); float s = height / 12.0;
    float left = sin(phase); float right = sin(phase + PI);
    float bounce = -0.12 * s * abs(sin(2 * phase)); float sway = 0.10 * s * sin(phase);

    p[23].set(-0.65*s+sway, 1.55*s+bounce, 0.18*s*left); p[24].set(0.65*s+sway, 1.55*s+bounce, 0.18*s*right);
    p[11].set(-1.2*s, -1.5*s+bounce*0.6, -0.1*s*left); p[12].set(1.2*s, -1.5*s+bounce*0.6, -0.1*s*right);
    p[0].set(0.04*s*sin(phase), -3.45*s+bounce*0.4, 0);

    float lk = 0.32 * s * max(0, left); float rk = 0.32 * s * max(0, right);
    p[25].set(-0.65*s, 3.55*s - lk, 0.95*s*left); p[27].set(-0.65*s, 5.5*s - 0.08*s*max(0,left), 1.55*s*left);
    p[26].set(0.65*s, 3.55*s - rk, 0.95*s*right); p[28].set(0.65*s, 5.5*s - 0.08*s*max(0,right), 1.55*s*right);
    
    p[13].set(-1.52*s, -0.05*s, -0.65*s*left); p[15].set(-1.62*s, 1.35*s, -1.1*s*left);
    p[14].set(1.52*s, -0.05*s, -0.65*s*right); p[16].set(1.62*s, 1.35*s, -1.1*s*right);
    syncFingers(p); syncFeet(p);
  }

  void syncFingers(PVector[] p) {
    float s = height / 12.0;
    p[17].set(p[15].x - 0.2*s, p[15].y + 0.04*s, p[15].z + 0.05*s);
    p[19].set(p[15].x - 0.05*s, p[15].y + 0.02*s, p[15].z + 0.13*s);
    p[21].set(p[15].x + 0.1*s, p[15].y - 0.02*s, p[15].z + 0.1*s);
    p[18].set(p[16].x + 0.2*s, p[16].y + 0.04*s, p[16].z + 0.05*s);
    p[20].set(p[16].x + 0.05*s, p[16].y + 0.02*s, p[16].z + 0.13*s);
    p[22].set(p[16].x - 0.1*s, p[16].y - 0.02*s, p[16].z + 0.1*s);
  }

  void syncFeet(PVector[] p) {
    float s = height / 12.0;
    p[29].set(p[27].x, p[27].y + 0.18*s, p[27].z - 0.16*s);
    p[31].set(p[27].x, p[27].y + 0.12*s, p[27].z + 0.48*s);
    p[30].set(p[28].x, p[28].y + 0.18*s, p[28].z - 0.16*s);
    p[32].set(p[28].x, p[28].y + 0.12*s, p[28].z + 0.48*s);
  }
}

// ================================================================
// OBJECT: DATA STORE (Static encyclopedic definitions)
// ================================================================
static class DataStore {

  static final String[] categoryNames = {
    "TIGER (🐅)", "CRANE (🐦)", "SNAKE (🐍)", "MONKEY (🐒)", "EAGLE (🦅)", 
    "DRAGON (🐉)", "TAOLU CORE", "DRUNKEN (🍶)", "FROG (🐸)"
  };

  static final String[][] techniques = {
    { "Tiger Fist", "Tiger Claw", "Tiger Paw", "Tiger Advance", "Tiger Retreat", "Tiger Press", "Tiger Guard", "Tiger Sweep", "Tiger Crouch", "Tiger Leap" },
    { "Crane Stance", "Crane One-Leg", "Crane Wing", "Crane Guard", "Crane Step", "Crane Turn", "Crane Rise", "Crane Balance", "Crane Pivot", "Crane Jump" },
    { "Snake Hand", "Snake Finger", "Snake Glide", "Snake Retreat", "Snake Weave", "Snake Evade", "Snake Circle", "Snake Spiral", "Snake Coil", "Snake Flow" },
    { "Monkey Stance", "Monkey Crouch", "Monkey Hop", "Monkey Step", "Monkey Retreat", "Monkey Angle", "Monkey Turn", "Monkey Feint", "Monkey Roll", "Monkey Sequence" },
    { "Eagle Stance", "Eagle Wing", "Eagle Spread", "Eagle Reach", "Eagle Seize", "Eagle Intercept", "Eagle Retreat", "Eagle Pivot", "Eagle Descend", "Eagle Jump" },
    { "Dragon Stance", "Dragon Step", "Dragon Turn", "Dragon Spiral", "Dragon Circle", "Dragon Coil", "Dragon Rise", "Dragon Change", "Dragon Evade", "Dragon Entry" },
    { "Gongbu (Bow)", "Mabu (Horse)", "Xubu (Empty)", "Pubu (Drop)", "Dulibu (Single)", "Forward Entry", "Lateral Evade", "High Guard", "Intercept", "Recovery" },
    { "Drunken Stance", "Cup Holding", "Stagger Step", "Falling Feint", "Swaying Willow", "Ground Roll", "Deceptive Strike", "Off-balance Pivot", "Jug Drink", "Drunken Retreat" },
    { "Frog Stance", "Deep Crouch", "Leaping Strike", "Ground Press", "Frog Bound", "Low Sweep", "Amphibian Guard", "Spring Load", "Lateral Hop", "Water Entry" }
  };

  static final String[][] trainingType = {
    { "Hand velocity", "Finger configuration", "Impulse measurement", "COM velocity", "Deceleration control", "Forward force vector", "Structural geometry", "Foot trajectory", "COM height drop", "Take-off velocity" },
    { "Postural balance", "COP displacement", "Arm trajectory", "Shoulder geometry", "Foot placement precision", "Angular velocity", "COM elevation", "Stability duration", "Foot rotation", "Take-off velocity" },
    { "Hand trajectory", "Finger/palm shape", "Continuous velocity", "COM displacement", "Lateral motion", "Reaction evasion", "Trajectory curvature (κ)", "Angular velocity", "Stored joint configuration", "Motion smoothness" },
    { "COM variance", "Level change velocity", "Vertical hop velocity", "Foot trajectory", "Displacement unpredictability", "Direction change rate", "Angular velocity", "Prediction error", "Body orientation", "Sequence irregularity" },
    { "Balance control", "Arm sweep arc", "Shoulder extension", "End-effector range", "Hand trajectory", "Reaction time (T-intercept)", "Distance control", "Foot angle", "Vertical velocity", "Take-off impulse" },
    { "Postural geometry", "COM trajectory", "Trunk angular velocity", "Curvature analysis", "Circular trajectory", "Joint twist configuration", "Vertical displacement", "Direction variation", "Lateral velocity", "Relative distance entry" },
    { "Front/rear mass loading", "Base width stability", "Weight distribution (rear)", "Low COM / hip mobility", "Single-leg COP stability", "Linear COM velocity", "Lateral displacement", "Defensive coverage", "Response timing", "Return-to-base time" },
    { "Off-axis COM", "Momentum redirection", "Asymmetrical balance", "Faux-fall recovery", "Rotational inertia", "Spinal flexion", "Unpredictable timing", "Deceptive targeting", "Base instability", "Recovery impulse" },
    { "Deep kinetic loading", "Quad extension rate", "Ground reaction force", "Vertical displacement", "Quadrilateral stance", "Hip mobility", "Spring mechanics", "Low-profile evasion", "Explosive thrust", "Landing absorption" }
  };

  static final int[][] connections = {
    {0,1}, {1,2}, {2,3}, {3,7}, {0,4}, {4,5}, {5,6}, {6,8}, {9,10}, 
    {0,11}, {0,12}, 
    {11,12}, {11,23}, {12,24}, {23,24}, 
    {11,13}, {13,15}, {15,17}, {15,19}, {15,21}, {17,19}, 
    {12,14}, {14,16}, {16,18}, {16,20}, {16,22}, {18,20}, 
    {23,25}, {25,27}, {27,29}, {27,31}, {29,31}, 
    {24,26}, {26,28}, {28,30}, {28,32}, {30,32} 
  };
}
