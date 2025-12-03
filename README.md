<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>วงล้อประจำตัวอาฉีลี่ 💜</title>
<style>
    body {
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        text-align: center;
        background: linear-gradient(135deg, #1d1b2e 0%, #2d1b3d 100%);
        color: white;
        padding: 20px;
        min-height: 100vh;
        margin: 0;
    }
    h1 {
        font-size: 2.5em;
        margin: 20px 0;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
    }
    p {
        font-size: 1.2em;
        color: #d9b3ff;
    }
    #wheelContainer {
        position: relative;
        display: inline-block;
        margin-top: 20px;
    }
    #pointer {
        position: absolute;
        top: -30px;
        left: 50%;
        transform: translateX(-50%);
        width: 0;
        height: 0;
        border-left: 20px solid transparent;
        border-right: 20px solid transparent;
        border-top: 35px solid #ffd700;
        filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        z-index: 10;
    }
    canvas {
        border-radius: 50%;
        box-shadow: 0 8px 30px rgba(160, 32, 240, 0.5);
    }
    button {
        padding: 15px 40px;
        margin-top: 30px;
        font-size: 20px;
        font-weight: bold;
        border: none;
        border-radius: 25px;
        background: linear-gradient(135deg, #a020f0 0%, #d946ef 100%);
        color: white;
        cursor: pointer;
        transition: all 0.3s;
        box-shadow: 0 4px 15px rgba(160, 32, 240, 0.4);
    }
    button:hover {
        transform: translateY(-2px);
        box-shadow: 0 6px 20px rgba(160, 32, 240, 0.6);
    }
    button:active {
        transform: scale(0.95);
    }
    button:disabled {
        opacity: 0.6;
        cursor: not-allowed;
    }
    #result {
        margin-top: 30px;
        font-size: 1.8em;
        min-height: 60px;
        color: #ffd700;
        text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        animation: fadeIn 0.5s;
    }
    #spinCount {
        margin-top: 10px;
        font-size: 1.3em;
        color: #d9b3ff;
    }
    .spin-limit-reached {
        color: #ff6b6b;
        font-weight: bold;
    }
    @keyframes fadeIn {
        from { opacity: 0; transform: translateY(10px); }
        to { opacity: 1; transform: translateY(0); }
    }
</style>
</head>
<body>

<h1>💜 วงล้อประจำตัวอาฉีลี่ 💜</h1>
<p>หมุนล้อเพื่อสุ่มคำทำนายวันนี้!</p>

<div id="wheelContainer">
    <div id="pointer"></div>
    <canvas id="wheelCanvas" width="400" height="400"></canvas>
</div>
<br>
<button id="spinBtn" onclick="spinWheel()">หมุนเลย! ✨</button>

<div id="spinCount">เหลือโอกาสหมุน: <span id="remainingSpins">3</span>/3 ครั้ง</div>
<h2 id="result"></h2>

<script>
    const options = [
        "โชคดีมาก 💜",
        "เอา",
        "ได้เงินกะทันหัน 💸",
        "ไม่เอา",
        "มีคนคิดถึงคุณ ✨",
        "พักไว้พรุ่งนี้",
        "มีโอกาสใหม่เข้ามา",
        "ได้ไอเดียดี ๆ 💡",
        "ไป",
        "มีเสน่ห์เป็นพิเศษ 💜",
        "ไม่ไป",
        "ระวังเรื่องเงินนิดนึง 💰",
        "พักก่อน",
        "มีเรื่องดี ๆ จากครอบครัว 🏠"
    ];

    const MAX_SPINS = 3;
    let spinsRemaining = MAX_SPINS;

    const canvas = document.getElementById("wheelCanvas");
    const ctx = canvas.getContext("2d");
    const size = canvas.width / 2;
    const arc = (2 * Math.PI) / options.length;
    let angle = 0;
    let spinning = false;

    const colors = ["#b366ff", "#d9b3ff", "#9d4edd", "#c77dff", "#b088f9", "#da9ff9"];

    // ฟังก์ชันสำหรับดึงวันที่ปัจจุบัน (YYYY-MM-DD)
    function getCurrentDate() {
        const now = new Date();
        return now.toISOString().split('T')[0];
    }

    // โหลดข้อมูลจำนวนครั้งที่หมุนจาก LocalStorage
    function loadSpinData() {
        try {
            const stored = localStorage.getItem('achille-wheel-data');
            if (stored) {
                const data = JSON.parse(stored);
                const today = getCurrentDate();
                
                // ถ้าวันที่เดียวกัน ใช้ข้อมูลเดิม ถ้าต่างวัน รีเซ็ต
                if (data.date === today) {
                    spinsRemaining = data.remaining;
                } else {
                    spinsRemaining = MAX_SPINS;
                    saveSpinData();
                }
            } else {
                spinsRemaining = MAX_SPINS;
                saveSpinData();
            }
        } catch (error) {
            console.log('ไม่สามารถโหลดข้อมูลได้ ใช้ค่าเริ่มต้น');
            spinsRemaining = MAX_SPINS;
        }
        updateSpinCount();
    }

    // บันทึกข้อมูลจำนวนครั้งที่หมุนลง LocalStorage
    function saveSpinData() {
        try {
            const data = {
                date: getCurrentDate(),
                remaining: spinsRemaining
            };
            localStorage.setItem('achille-wheel-data', JSON.stringify(data));
        } catch (error) {
            console.log('ไม่สามารถบันทึกข้อมูลได้');
        }
    }

    function updateSpinCount() {
        const spinCountElement = document.getElementById("remainingSpins");
        const spinCountContainer = document.getElementById("spinCount");
        spinCountElement.innerText = spinsRemaining;
        
        if (spinsRemaining === 0) {
            spinCountContainer.classList.add("spin-limit-reached");
            spinCountElement.innerText = "0";
        } else {
            spinCountContainer.classList.remove("spin-limit-reached");
        }
    }

    function drawWheel() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        
        for (let i = 0; i < options.length; i++) {
            const start = angle + i * arc;
            ctx.beginPath();
            ctx.fillStyle = colors[i % colors.length];
            ctx.moveTo(size, size);
            ctx.arc(size, size, size - 10, start, start + arc);
            ctx.fill();
            ctx.strokeStyle = "#fff";
            ctx.lineWidth = 2;
            ctx.stroke();

            ctx.save();
            ctx.translate(size, size);
            ctx.rotate(start + arc / 2);
            ctx.textAlign = "right";
            ctx.fillStyle = "#000";
            ctx.font = "bold 14px sans-serif";
            ctx.fillText(options[i], size - 20, 5);
            ctx.restore();
        }

        // Center circle
        ctx.beginPath();
        ctx.fillStyle = "#ffd700";
        ctx.arc(size, size, 30, 0, 2 * Math.PI);
        ctx.fill();
        ctx.strokeStyle = "#fff";
        ctx.lineWidth = 3;
        ctx.stroke();
    }

    function spinWheel() {
        if (spinning) return;
        
        if (spinsRemaining <= 0) {
            document.getElementById("result").innerText = "หมดโอกาสหมุนวันนี้แล้ว! กลับมาใหม่พรุ่งนี้นะ 💜";
            return;
        }

        spinning = true;
        document.getElementById("spinBtn").disabled = true;
        document.getElementById("result").innerText = "";

        spinsRemaining--;
        updateSpinCount();
        saveSpinData();

        // สุ่มความเร็วเริ่มต้นที่แตกต่างกันมาก (0.3 - 0.8)
        let speed = Math.random() * 0.5 + 0.3;
        // สุ่มอัตราการชะลอที่แตกต่างกัน (0.993 - 0.998)
        const slow = Math.random() * 0.005 + 0.993;
        const minSpeed = 0.001;

        const spin = setInterval(() => {
            angle += speed;
            speed *= slow;

            drawWheel();

            if (speed < minSpeed) {
                clearInterval(spin);
                spinning = false;
                
                if (spinsRemaining > 0) {
                    document.getElementById("spinBtn").disabled = false;
                }

                // คำนวณผลลัพธ์จากตำแหน่งลูกศรที่ชี้ (ด้านบนตรงกลาง = 270 องศา หรือ -90 องศา)
                const pointerAngle = (3 * Math.PI / 2); // 270 degrees (ด้านบน)
                const normalizedAngle = (pointerAngle - angle) % (2 * Math.PI);
                const positiveAngle = (normalizedAngle + 2 * Math.PI) % (2 * Math.PI);
                const index = Math.floor(positiveAngle / arc);
                const result = options[index];

                setTimeout(() => {
                    document.getElementById("result").innerText = "ผลลัพธ์: " + result;
                }, 100);
            }
        }, 16);
    }

    // เริ่มต้นโหลดข้อมูล
    loadSpinData();
    drawWheel();
</script>

</body>
</html>
