<div align="center">
  <style>
    .pyramid-container {
      margin: 20px auto;
      max-width: 800px;
    }
    .pyramid-step {
      margin: 10px auto;
      padding: 15px 20px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      transition: transform 0.2s;
    }
    .pyramid-step:hover {
      transform: translateY(-2px);
      box-shadow: 0 4px 8px rgba(0,0,0,0.15);
    }
    .step-1 {
      width: 80%;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
    }
    .step-2 {
      width: 65%;
      background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
      color: white;
    }
    .step-3 {
      width: 50%;
      background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
      color: white;
    }
    .step-title {
      font-weight: bold;
      font-size: 1.1em;
      margin-bottom: 5px;
    }
    .step-detail {
      font-size: 0.95em;
      opacity: 0.9;
    }
  </style>
  
  <div class="pyramid-container">
    <h3>Progress So Far</h3>
    
    <div class="pyramid-step step-1">
      <div class="step-title">Step 1: Proposed initial idea</div>
      <div class="step-detail">(First Review)</div>
    </div>
    
    <div class="pyramid-step step-2">
      <div class="step-title">Step 2: Refined scope via forum discussion</div>
      <div class="step-detail">to align with GSoC 2026 timeline (Second Review)</div>
    </div>
    
    <div class="pyramid-step step-3">
      <div class="step-title">Step 3: Map complete end-to-end deliverables</div>
      <div class="step-detail">for selected project(s) [Pending]</div>
    </div>
  </div>
</div>
