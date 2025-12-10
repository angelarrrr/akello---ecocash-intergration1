import streamlit as st
import random
from datetime import date, timedelta

# ----------------------------
# PAGE CONFIG
# ----------------------------
st.set_page_config(page_title="Akello Learning", layout="wide", page_icon="📘")

st.markdown(
    """
    <style>
    .stApp {
        background: linear-gradient(135deg, #ffd6e0, #e0bbe4, #d4c1ec);
    }
    .block-container { padding-top: 1rem; }
    .stButton>button {
        border-radius: 16px !important;
        font-weight: 900 !important;
        border-width: 3px !important;
        box-shadow: 0 4px 12px rgba(0,0,0,0.2) !important;
        transition: transform 0.2s !important;
    }
    .stButton>button:hover {
        transform: scale(1.03) !important;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# ----------------------------
# STATE INITIALIZATION
# ----------------------------
if "user" not in st.session_state:
    st.session_state.user = {
        "name": "John Doe",
        "xp": 1600,  # Rhinoceros level
        "ecocash_balance": 15.50,
        "is_premium": False,
        "contributions": 12,
        "streak": 5,
        "last_login": date.today().isoformat(),
    }

if "purchased_materials" not in st.session_state:
    st.session_state.purchased_materials = []

if "active_tab" not in st.session_state:
    st.session_state.active_tab = "marketplace"

if "show_ad" not in st.session_state:
    st.session_state.show_ad = not st.session_state.user["is_premium"]

# Payment modal state
if "show_payment_modal" not in st.session_state:
    st.session_state.show_payment_modal = False
    st.session_state.selected_material = None
    st.session_state.payment_form = {
        "phone": "",
        "pin": "",
        "confirm_pin": "",
        "confirm_payment": False
    }

# Voucher claim state
if "voucher_claimed" not in st.session_state:
    st.session_state.voucher_claimed = False
if "claim_phone" not in st.session_state:
    st.session_state.claim_phone = ""

# UPLOAD & CHAT STATE
if "show_upload_modal" not in st.session_state:
    st.session_state.show_upload_modal = False

if "new_chat_message" not in st.session_state:
    st.session_state.new_chat_message = ""

if "user_chat_id" not in st.session_state:
    st.session_state.user_chat_id = 100

# NEW: Answering state
if "answering_message_id" not in st.session_state:
    st.session_state.answering_message_id = None

# Chat messages
chat_messages = [
    {"id": 1, "user": "Sarah M.", "msg": "Can someone explain quadratic equations?", "xp": 15, "replies": 3},
    {"id": 2, "user": "Mike T.", "msg": "What's the best way to memorize periodic table?", "xp": 10, "replies": 5},
]

# ----------------------------
# DATA
# ----------------------------
materials = [
    {"id": 1, "title": "Advanced Mathematics Notes", "price": 2.50, "seller": "Prof. Moyo", "sales": 45},
    {"id": 2, "title": "Chemistry Revision Pack", "price": 3.00, "seller": "Dr. Ncube", "sales": 32},
    {"id": 3, "title": "Physics Formula Guide", "price": 1.50, "seller": "Mr. Chikwanha", "sales": 67},
]

leaderboard = [
    {"rank": 1, "name": "Tendai K.", "xp": 4850, "badge": "Elephant"},
    {"rank": 2, "name": "Rumbi M.", "xp": 4200, "badge": "Elephant"},
    {"rank": 3, "name": "Tapiwa N.", "xp": 3800, "badge": "Elephant"},
    {"rank": 4, "name": "John Doe", "xp": 1600, "badge": "Rhinoceros"},
    {"rank": 5, "name": "Blessing S.", "xp": 1200, "badge": "Buffalo"},
]

ttal_questions = [
    {
        "statements": [
            "Zimbabwe gained independence from Britain in 1980.",
            "Mount Nyangani is the highest mountain in Zimbabwe.",
            "Zimbabwe's capital is Bulawayo."
        ],
        "lie_index": 2
    },
    {
        "statements": [
            "The Great Zimbabwe ruins are a UNESCO World Heritage Site.",
            "Zimbabwe has 16 official languages.",
            "Lake Kariba is the smallest man-made lake in the world."
        ],
        "lie_index": 2
    },
    {
        "statements": [
            "Robert Mugabe was Zimbabwe's first president after independence.",
            "Chimanimani Mountains are in the west of Zimbabwe.",
            "The Zimbabwe Bird is a national symbol featured on the flag."
        ],
        "lie_index": 1
    },
    {
        "statements": [
            "Hwange National Park is Zimbabwe's largest game reserve.",
            "The Zambezi River forms the border between Zimbabwe and Zambia.",
            "Zimbabwe is located in West Africa."
        ],
        "lie_index": 2
    },
    {
        "statements": [
            "Nyanga is known as the 'Switzerland of Zimbabwe'.",
            "The currency of Zimbabwe is the Zimbabwean Dollar (ZWL).",
            "Zimbabwe shares a border with Tanzania."
        ],
        "lie_index": 2
    },
    {
        "statements": [
            "Dr. Solomon Mujuru was a famous musician.",
            "Victoria Falls is one of the Seven Natural Wonders of the World.",
            "The mbira is a traditional Zimbabwean musical instrument."
        ],
        "lie_index": 0
    }
]

BADGE_CONFIG = {
    "Elephant": {"min_xp": 5000, "icon": "🐘", "gradient": "linear-gradient(135deg, #8B5CF6, #EC4899)", "border": "#8B5CF6"},
    "Rhinoceros": {"min_xp": 1500, "icon": "🦏", "gradient": "linear-gradient(135deg, #9CA3AF, #4B5563)", "border": "#6B7280"},
    "Buffalo": {"min_xp": 1000, "icon": "🐃", "gradient": "linear-gradient(135deg, #F97316, #EF4444)", "border": "#F97316"},
    "Lion": {"min_xp": 500, "icon": "🦁", "gradient": "linear-gradient(135deg, #FACC15, #F97316)", "border": "#F59E0B"},
    "Leopard": {"min_xp": 0, "icon": "🐆", "gradient": "linear-gradient(135deg, #0D9488, #0891B2)", "border": "#0D9488"},
}

# ----------------------------
# HELPERS
# ----------------------------
def get_badge(xp):
    for name, config in BADGE_CONFIG.items():
        if xp >= config["min_xp"]:
            return {"name": name, **config}
    return BADGE_CONFIG["Leopard"]

def add_xp(amount):
    st.session_state.user["xp"] += amount

def claim_daily():
    today = date.today()
    last_login = date.fromisoformat(st.session_state.user["last_login"])
    if last_login == today:
        st.warning("✅ You already claimed your attendance today!")
        return
    if last_login == today - timedelta(days=1):
        st.session_state.user["streak"] += 1
    else:
        st.session_state.user["streak"] = 1
    streak = st.session_state.user["streak"]
    bonus = 5
    if streak >= 30:
        bonus += 20
    elif streak >= 14:
        bonus += 10
    elif streak >= 7:
        bonus += 5
    add_xp(bonus)
    st.session_state.user["last_login"] = today.isoformat()
    st.success(f"🎉 Day {streak} Streak! +{bonus} XP earned!")

def open_payment_modal(material):
    st.session_state.show_payment_modal = True
    st.session_state.selected_material = material
    st.session_state.payment_form = {
        "phone": "",
        "pin": "",
        "confirm_pin": "",
        "confirm_payment": False
    }

def process_purchase():
    form = st.session_state.payment_form
    material = st.session_state.selected_material
    if not form["phone"]:
        st.error("❌ Please enter your EcoCash phone number!")
        return
    if len(form["phone"]) < 10:
        st.error("❌ Please enter a valid 10-digit phone number!")
        return
    if not form["pin"]:
        st.error("❌ Please enter your EcoCash PIN!")
        return
    if len(form["pin"]) != 4 or not form["pin"].isdigit():
        st.error("❌ PIN must be 4 digits!")
        return
    if form["pin"] != form["confirm_pin"]:
        st.error("❌ PINs do not match!")
        return
    if not form["confirm_payment"]:
        st.error("❌ Please confirm your payment!")
        return
    if st.session_state.user["ecocash_balance"] < material["price"]:
        st.error("❌ Insufficient EcoCash balance.")
        return
    st.session_state.user["ecocash_balance"] -= material["price"]
    st.session_state.purchased_materials.append(material["id"])
    add_xp(5)
    st.session_state.show_payment_modal = False
    st.success(f'✅ Purchased "{material["title"]}"! +5 XP earned!')

def claim_voucher():
    if not st.session_state.claim_phone:
        st.error("❌ Please enter your phone number to receive the voucher!")
        return
    if len(st.session_state.claim_phone) < 10:
        st.error("❌ Please enter a valid 10-digit phone number!")
        return
    st.session_state.voucher_claimed = True
    st.success(f"🎉 Voucher claimed! $1 will be sent to {st.session_state.claim_phone} within 24 hours!")

def upload_material(title, price, description):
    if not title or not price or not description:
        st.error("❌ Please fill all fields!")
        return
    try:
        price = float(price)
        if price <= 0:
            st.error("❌ Price must be greater than $0!")
            return
    except:
        st.error("❌ Invalid price!")
        return
    st.session_state.show_upload_modal = False
    st.success(f"✅ Uploaded '{title}' for ${price}! It will appear in the marketplace shortly.")

# ----------------------------
# HEADER
# ----------------------------
user = st.session_state.user
badge = get_badge(user["xp"])

st.markdown(
    f"""
    <div style="background: linear-gradient(to right, #FF6B6B, #A052D9, #20C997); 
                padding: 1.5rem; border-radius: 24px; margin-bottom: 1rem; box-shadow: 0 8px 24px rgba(0,0,0,0.3);">
        <h1 style="color: white; font-weight: 900; font-size: 2.8rem; margin: 0;">📘 Akello Learning</h1>
    </div>
    """,
    unsafe_allow_html=True,
)

cols = st.columns(3)
with cols[0]:
    st.markdown(
        f"""
        <div style="background: linear-gradient(135deg, #34D399, #10B981); padding: 16px; border-radius: 20px;
                    text-align: center; border: 4px solid #059669; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
            <div style="font-size: 14px; font-weight: bold; color: #064E3B;">💰 EcoCash Balance</div>
            <div style="font-size: 32px; font-weight: 900; color: #064E3B;">${user['ecocash_balance']:.2f}</div>
        </div>
        """,
        unsafe_allow_html=True,
    )
with cols[1]:
    st.markdown(
        f"""
        <div style="background: linear-gradient(135deg, #F59E0B, #EA580C); padding: 16px; border-radius: 20px;
                    text-align: center; border: 4px solid #C2410C; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
            <div style="font-size: 14px; font-weight: bold; color: #9A3412;">🔥 Streak</div>
            <div style="font-size: 32px; font-weight: 900; color: #9A3412;">{user['streak']} Days</div>
        </div>
        """,
        unsafe_allow_html=True,
    )
with cols[2]:
    st.markdown(
        f"""
        <div style="background: {badge['gradient']}; padding: 16px; border-radius: 20px; text-align: center;
                    border: 4px solid {badge['border']}; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
            <div style="font-size: 42px; margin-bottom: 8px;">{badge['icon']}</div>
            <div style="font-size: 14px; font-weight: bold; color: white;">{user['name']}</div>
            <div style="font-size: 14px; font-weight: bold; color: white;">{user['xp']} XP • {badge['name']}</div>
        </div>
        """,
        unsafe_allow_html=True,
    )

# ----------------------------
# AD BANNER
# ----------------------------
if st.session_state.show_ad:
    colA, colB, colC = st.columns([1, 4, 1])
    with colB:
        st.markdown(
            """
            <div style="background: linear-gradient(to right, #FBBF24, #F59E0B); 
                        padding: 12px; border-radius: 16px; text-align: center; 
                        border: 3px solid #D97706; box-shadow: 0 4px 12px rgba(0,0,0,0.3);">
                <span style="color: #78350F; font-weight: 900; font-size: 18px;">
                    ✨ Upgrade to Premium for ad-free experience! Only $1/month ✨
                </span>
            </div>
            """,
            unsafe_allow_html=True,
        )
    with colC:
        if st.button("❌", key="close_ad"):
            st.session_state.show_ad = False
    st.divider()

# ----------------------------
# TABS
# ----------------------------
tabs = ["Marketplace", "Chat", "Game", "Streaks", "Leaderboard", "Badges", "Premium"]
tab_cols = st.columns(len(tabs))
for i, tab in enumerate(tabs):
    with tab_cols[i]:
        if st.button(tab, key=f"tab_{tab}", use_container_width=True):
            st.session_state.active_tab = tab.lower()

active_tab = st.session_state.active_tab

# ----------------------------
# MARKETPLACE
# ----------------------------
if active_tab == "marketplace":
    st.markdown('<h2 style="color: #7C3AED; font-weight: 900;">📚 Learning Materials</h2>', unsafe_allow_html=True)
    if st.button("📤 Upload Material", use_container_width=True):
        st.session_state.show_upload_modal = True
    cols = st.columns(3)
    for i, mat in enumerate(materials):
        with cols[i]:
            owned = mat["id"] in st.session_state.purchased_materials
            if owned:
                st.markdown(
                    '<div style="background: linear-gradient(135deg, #34D399, #10B981); color: white; '
                    'padding: 8px; border-radius: 12px; font-weight: bold; text-align: center; '
                    'border: 2px solid #059669; margin-bottom: 10px;">✅ OWNED</div>',
                    unsafe_allow_html=True,
                )
            st.subheader(mat["title"])
            st.caption(f"By {mat['seller']}")
            st.write(f"👥 {mat['sales']} students purchased")
            st.markdown(f"### ${mat['price']}")
            if owned:
                st.button("Purchased ✓", disabled=True, key=f"btn_{mat['id']}", use_container_width=True)
            else:
                if st.button("Buy Now!", key=f"buy_{mat['id']}", use_container_width=True):
                    open_payment_modal(mat)

# ----------------------------
# CHAT – NO "REPLY" BUTTONS (ONLY "ANSWER →")
# ----------------------------
elif active_tab == "chat":
    st.markdown('<h2 style="color: #0D9488; font-weight: 900;">💬 Community Chat</h2>', unsafe_allow_html=True)
    chatbot_uses = "∞" if user["is_premium"] else "3"
    st.info(f"💬 **Chatbot uses remaining**: {chatbot_uses}")
    st.caption("Earn XP by helping others! AI evaluates response quality.")

    # Post your question
    st.markdown("### ❓ Ask a Question")
    new_msg = st.text_area("Type your question here...", key="chat_input", value=st.session_state.new_chat_message, height=100)
    if st.button("📤 Post Question", use_container_width=True):
        if new_msg.strip():
            st.session_state.new_chat_message = ""
            st.session_state.user_chat_id += 1
            add_xp(10)
            st.success("✅ Posted! +10 XP earned!")
        else:
            st.error("❌ Please type a message!")

    st.markdown("### 💬 Recent Messages")
    for msg in chat_messages:
        with st.container():
            st.markdown(f"**{msg['user']}**")
            st.write(msg["msg"])
            col1, col2 = st.columns([4, 1])
            with col2:
                st.markdown(f"⚡ +{msg['xp']} XP")
            # Show number of replies (like React), but NO "Reply" button
            st.caption(f"{msg['replies']} replies")
            
            # ONLY show "Answer →" for user to engage
            if st.button("Answer →", key=f"answer_{msg['id']}", use_container_width=True):
                st.session_state.answering_message_id = msg["id"]
            
            # Answer input
            if st.session_state.answering_message_id == msg["id"]:
                answer = st.text_area(
                    f"Your answer to {msg['user']}:", 
                    key=f"ans_text_{msg['id']}",
                    height=100
                )
                if st.button("📤 Submit Answer", key=f"submit_ans_{msg['id']}", use_container_width=True):
                    if not answer.strip():
                        st.error("❌ Please provide a real answer!")
                    else:
                        keywords = ["solve", "remember", "practice", "tip", "method", "example", "learn", "study"]
                        is_good = any(kw in answer.lower() for kw in keywords)
                        add_xp(msg["xp"])
                        if is_good:
                            st.success(f"✅ Great answer! +{msg['xp']} XP earned!")
                        else:
                            st.success(f"✅ Thanks for helping! +{msg['xp']} XP earned!")
                        st.session_state.answering_message_id = None
                        st.rerun()
                if st.button("❌ Cancel", key=f"cancel_ans_{msg['id']}", use_container_width=True):
                    st.session_state.answering_message_id = None
                    st.rerun()
            
            st.divider()

    if st.button("Simulate Correct Answer (+10 XP)"):
        add_xp(10)
        st.success("✅ Correct! +10 XP earned!")

# ----------------------------
# GAME – TWO TRUTHS AND A LIE
# ----------------------------
elif active_tab == "game":
    st.markdown('<h2 style="color: #7C3AED; font-weight: 900;">🧐 Two Truths and a Lie: Zimbabwe Edition!</h2>', unsafe_allow_html=True)
    
    if "ttal" not in st.session_state:
        st.session_state.ttal = {
            "score": 0,
            "streak": 0,
            "current_question": None,
            "feedback": None,
            "game_won": False,
            "total_played": 0
        }
    
    game = st.session_state.ttal
    WIN_STREAK = 5

    cols = st.columns(3)
    with cols[0]:
        st.markdown(
            f"""
            <div style="background: linear-gradient(135deg, #FBBF24, #F59E0B); padding: 16px; border-radius: 20px;
                        text-align: center; border: 4px solid #D97706; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
                <div style="font-size: 14px; font-weight: bold; color: #78350F;">✅ Correct</div>
                <div style="font-size: 32px; font-weight: 900; color: #78350F;">{game['score']}</div>
            </div>
            """,
            unsafe_allow_html=True,
        )
    with cols[1]:
        st.markdown(
            f"""
            <div style="background: linear-gradient(135deg, #8B5CF6, #A78BFA); padding: 16px; border-radius: 20px;
                        text-align: center; border: 4px solid #7C3AED; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
                <div style="font-size: 14px; font-weight: bold; color: #5B21B6;">🔥 Streak</div>
                <div style="font-size: 32px; font-weight: 900; color: #5B21B6;">{game['streak']}</div>
            </div>
            """,
            unsafe_allow_html=True,
        )
    with cols[2]:
        st.markdown(
            f"""
            <div style="background: linear-gradient(135deg, #10B981, #059669); padding: 16px; border-radius: 20px;
                        text-align: center; border: 4px solid #047857; box-shadow: 0 6px 16px rgba(0,0,0,0.3);">
                <div style="font-size: 14px; font-weight: bold; color: #064E3B;">🎯 Goal</div>
                <div style="font-size: 32px; font-weight: 900; color: #064E3B;">{WIN_STREAK} in a Row!</div>
            </div>
            """,
            unsafe_allow_html=True,
        )

    st.progress(min(game["streak"] / WIN_STREAK, 1.0))

    if game["game_won"]:
        st.markdown(
            """
            <div style="background: linear-gradient(135deg, #10B981, #059669);
                        padding: 24px; border-radius: 24px; text-align: center;
                        border: 4px solid #047857; box-shadow: 0 12px 32px rgba(0,0,0,0.4); margin: 20px 0;">
                <div style="font-size: 80px; margin-bottom: 16px;">🏆</div>
                <h3 style="color: white; font-weight: 900; font-size: 36px; margin: 0;">ZIMBABWE EXPERT!</h3>
                <p style="color: #D1FAE5; font-weight: bold; margin-top: 12px; font-size: 20px;">+50 XP Bonus!</p>
            </div>
            """,
            unsafe_allow_html=True,
        )
        if st.button("🎯 Play Again!", use_container_width=True):
            st.session_state.ttal = {
                "score": 0,
                "streak": 0,
                "current_question": None,
                "feedback": None,
                "game_won": False,
                "total_played": 0
            }
        st.stop()

    if not game["current_question"] and not game["feedback"]:
        game["current_question"] = random.choice(ttal_questions)

    if game["current_question"]:
        st.markdown("### 🇿🇼 Which statement is **FALSE**?")
        q = game["current_question"]
        for i, stmt in enumerate(q["statements"]):
            disabled = game["feedback"] is not None
            if st.button(f"{i+1}. {stmt}", key=f"stmt_{i}", disabled=disabled, use_container_width=True):
                is_lie = (i == q["lie_index"])
                game["feedback"] = "correct" if is_lie else "wrong"
                st.rerun()

    if game["feedback"]:
        if game["feedback"] == "correct":
            st.success("✅ CORRECT! That was the lie!")
            add_xp(10)
            game["score"] += 1
            game["streak"] += 1
            game["total_played"] += 1
            if game["streak"] >= WIN_STREAK:
                add_xp(50)
                game["game_won"] = True
        else:
            st.error("❌ WRONG! That was actually true.")
            game["streak"] = 0
            game["total_played"] += 1
        if st.button("➡️ Next Question", use_container_width=True):
            game["current_question"] = None
            game["feedback"] = None
            st.rerun()

    with st.expander("📘 How to Play"):
        st.markdown("""
        - Read **three statements** about Zimbabwe.
        - **Two are TRUE**, **one is FALSE**.
        - Tap the statement you think is the **lie**.
        - Get **5 correct in a row** to win **+50 XP**!
        """)

# ----------------------------
# STREAKS
# ----------------------------
elif active_tab == "streaks":
    st.markdown('<h2 style="color: #F59E0B; font-weight: 900;">🔥 Daily Streaks</h2>', unsafe_allow_html=True)
    cols = st.columns(4)
    milestones = [(7, 5), (14, 10), (30, 20), (100, 50)]
    for days, xp in milestones:
        unlocked = user["streak"] >= days
        with cols[milestones.index((days, xp))]:
            st.markdown(
                f"<div style='text-align: center;'><div style='background: {'linear-gradient(135deg, #34D399, #10B981)' if unlocked else '#E5E7EB'}; color: {'white' if unlocked else '#4B5563'}; padding: 12px; border-radius: 16px; font-weight: bold;'>{'✅' if unlocked else '🔒'} **{days} Days</div><div style='font-size: 12px; color: #6B7280; margin-top: 4px;'>+{xp} XP Bonus</div></div>",
                unsafe_allow_html=True,
            )
    st.button("🔥 Claim Daily Attendance 🔥", on_click=claim_daily, use_container_width=True)

# ----------------------------
# LEADERBOARD
# ----------------------------
elif active_tab == "leaderboard":
    st.markdown('<h2 style="color: #7C3AED; font-weight: 900;">🏆 Monthly Leaderboard</h2>', unsafe_allow_html=True)
    st.info("Top 5 Win EcoCash Vouchers! 1st: $5 | 2nd: $4 | 3rd: $3 | 4th: $2 | 5th: $1")
    for entry in leaderboard:
        badge_icon = get_badge(entry["xp"])["icon"]
        is_user = entry["name"] == user["name"]
        st.markdown(
            f"""
            <div style="background: white; padding: 12px; border-radius: 16px; margin-bottom: 10px; 
                        border-left: 4px solid {'#F59E0B' if entry['rank'] <= 3 else '#6B7280'};">
                <div style="display: flex; justify-content: space-between; align-items: center;">
                    <div style="display: flex; align-items: center; gap: 12px;">
                        {"👑" if is_user else ""} #{entry['rank']} {badge_icon} <b>{entry['name']}</b> — {entry['xp']} XP
                    </div>
                    {f'<span>${6 - entry["rank"]} voucher</span>' if entry['rank'] <= 5 else ''}
                </div>
            </div>
            """,
            unsafe_allow_html=True,
        )
    user_rank = next((entry for entry in leaderboard if entry["name"] == user["name"]), None)
    if user_rank and user_rank["rank"] <= 5 and not st.session_state.voucher_claimed:
        st.markdown("### 📱 Claim Your Voucher")
        phone = st.text_input("Enter your EcoCash phone number (10 digits):", key="claim_phone_input", value=st.session_state.claim_phone)
        st.session_state.claim_phone = phone
        if st.button("📞 Claim Voucher", use_container_width=True):
            claim_voucher()
    elif st.session_state.voucher_claimed:
        st.success("✅ **Voucher already claimed!** Check your EcoCash inbox.")

# ----------------------------
# BADGES
# ----------------------------
elif active_tab == "badges":
    st.markdown('<h2 style="color: #0D9488; font-weight: 900;">🏅 Zimbabwe Big Five Badges</h2>', unsafe_allow_html=True)
    cols = st.columns(2)
    for i, (name, config) in enumerate(BADGE_CONFIG.items()):
        unlocked = user["xp"] >= config["min_xp"]
        with cols[i % 2]:
            st.markdown(
                f"""
                <div style="background: {'white' if not unlocked else config['gradient']}; 
                            padding: 20px; border-radius: 20px; margin-bottom: 16px;
                            border: 3px solid {config['border']}; box-shadow: 0 6px 16px rgba(0,0,0,0.2);">
                    <div style="display: flex; align-items: center; gap: 12px; margin-bottom: 12px;">
                        <span style="font-size: 42px;">{config['icon']}</span>
                        <div>
                            <div style="font-size: 24px; font-weight: 900; color: #374151;">{name}</div>
                            <div style="font-size: 16px; font-weight: bold; color: #6B7280;">{config['min_xp']}+ XP</div>
                        </div>
                    </div>
                    {'<div style="background: #34D399; color: #064E3B; padding: 6px; border-radius: 12px; font-weight: 900; text-align: center;">✅ UNLOCKED!</div>' 
                     if unlocked 
                     else f'<div style="background: #E5E7EB; color: #4B5563; padding: 6px; border-radius: 12px; font-weight: bold; text-align: center;">🔒 Need {config["min_xp"] - user["xp"]} more XP</div>'}
                </div>
                """,
                unsafe_allow_html=True,
            )

# ----------------------------
# PREMIUM
# ----------------------------
elif active_tab == "premium":
    st.markdown('<h2 style="color: #7C3AED; font-weight: 900;">⭐ Premium Features</h2>', unsafe_allow_html=True)
    cols = st.columns(3)
    features = [
        ("❌ No Ads", "Enjoy distraction-free learning!"),
        ("📚 Compilation Access", "Get super revision bundles!"),
        ("💬 Unlimited Chatbot", "Ask as many questions as you want!"),
    ]
    for i, (title, desc) in enumerate(features):
        with cols[i]:
            st.markdown(
                f"""
                <div style="background: white; padding: 20px; border-radius: 20px; border: 3px solid #7C3AED; box-shadow: 0 6px 16px rgba(0,0,0,0.2);">
                    <div style="font-size: 36px; text-align: center; margin-bottom: 12px;">{"❌" if i==0 else "📚" if i==1 else "💬"}</div>
                    <div style="font-weight: 900; text-align: center;">{title}</div>
                    <div style="text-align: center; color: #6B7280; font-size: 14px;">{desc}</div>
                </div>
                """,
                unsafe_allow_html=True,
            )
    st.markdown(
        """
        <div style="background: linear-gradient(135deg, #8B5CF6, #EC4899, #F59E0B); 
                    padding: 32px; border-radius: 24px; text-align: center;
                    border: 4px solid #FBBF24; box-shadow: 0 12px 32px rgba(0,0,0,0.4); margin-top: 24px;">
            <div style="font-size: 48px; margin-bottom: 16px;">👑</div>
            <div style="color: white; font-size: 28px; font-weight: 900; margin-bottom: 12px;">Only $1/month</div>
            <div style="color: #FFF9C4; font-weight: bold; font-size: 18px;">Unlock all premium features via EcoCash!</div>
        </div>
        """,
        unsafe_allow_html=True,
    )
    if st.button("🌟 Upgrade Now! 🌟", use_container_width=True):
        if user["ecocash_balance"] >= 1:
            user["ecocash_balance"] -= 1
            user["is_premium"] = True
            st.session_state.show_ad = False
            st.success("🌟 Premium activated!")
        else:
            st.error("❌ Insufficient balance.")

# ----------------------------
# MODALS
# ----------------------------
# Payment Modal
if st.session_state.show_payment_modal and st.session_state.selected_material:
    material = st.session_state.selected_material
    st.markdown(
        """
        <div style="background: white; padding: 24px; border-radius: 24px; box-shadow: 0 16px 40px rgba(0,0,0,0.3); max-width: 600px; margin: 20px auto; border: 4px solid #10B981;">
            <h3 style="color: #064E3B; font-weight: 900; text-align: center; font-size: 28px;">💳 EcoCash Payment</h3>
            <p style="text-align: center; color: #064E3B; font-weight: bold;">Complete your purchase securely</p>
        </div>
        """,
        unsafe_allow_html=True,
    )
    st.markdown("### 📦 Order Summary")
    st.write(f"**Item**: {material['title']}")
    st.write(f"**Seller**: {material['seller']}")
    st.write(f"**Total**: **${material['price']:.2f}**")
    phone = st.text_input("📱 EcoCash Phone Number (10 digits)", value=st.session_state.payment_form["phone"])
    pin = st.text_input("🔒 EcoCash PIN (4 digits)", type="password", value=st.session_state.payment_form["pin"])
    confirm_pin = st.text_input("✅ Confirm PIN", type="password", value=st.session_state.payment_form["confirm_pin"])
    confirm = st.checkbox("I confirm this payment", value=st.session_state.payment_form["confirm_payment"])
    st.session_state.payment_form["phone"] = phone
    st.session_state.payment_form["pin"] = pin
    st.session_state.payment_form["confirm_pin"] = confirm_pin
    st.session_state.payment_form["confirm_payment"] = confirm
    st.info(f"Your EcoCash Balance: **${user['ecocash_balance']:.2f}**")
    if user["ecocash_balance"] < material["price"]:
        st.error("⚠️ Insufficient balance")
    cols = st.columns(2)
    with cols[0]:
        if st.button("❌ Cancel", use_container_width=True):
            st.session_state.show_payment_modal = False
    with cols[1]:
        if st.button("✅ Pay Now", use_container_width=True):
            process_purchase()

# Upload Modal
if st.session_state.show_upload_modal:
    st.markdown(
        """
        <div style="background: white; padding: 24px; border-radius: 24px; box-shadow: 0 16px 40px rgba(0,0,0,0.3); max-width: 600px; margin: 20px auto; border: 4px solid #8B5CF6;">
            <h3 style="color: #7C3AED; font-weight: 900; text-align: center; font-size: 28px;">📤 Upload Learning Material</h3>
        </div>
        """,
        unsafe_allow_html=True,
    )
    title = st.text_input("📝 Material Title")
    price = st.number_input("💰 Price (USD)", min_value=0.0, step=0.5, format="%.2f")
    desc = st.text_area("📋 Description", height=100)
    uploaded_file = st.file_uploader("📁 Upload File (PDF, DOC, etc.)", type=["pdf", "doc", "docx", "txt"])
    cols = st.columns(2)
    with cols[0]:
        if st.button("❌ Cancel", use_container_width=True):
            st.session_state.show_upload_modal = False
    with cols[1]:
        if st.button("🚀 Upload!", use_container_width=True):
            upload_material(title, price, desc)

st.divider()
st.caption("Akello Learning Platform • Empowering Zimbabwean Students")
