#!/usr/bin/env python3
"""
Astro-Synthesis Chart Calculator
Uses Swiss Ephemeris (pyswisseph) for professional-grade accuracy.
Outputs Western + Vedic chart data ready for interpretation.

Usage:
    python calculate_chart.py --date "1994-11-19" --time "09:45" --place "Nagpur, India"
    python calculate_chart.py --date "1994-11-19" --time "09:45" --lat 21.1458 --lon 79.0882 --tz "Asia/Kolkata"
"""

import swisseph as swe
import argparse
import json
from datetime import datetime
import pytz
from geopy.geocoders import Nominatim
from timezonefinder import TimezoneFinder

# ── Constants ──────────────────────────────────────────────────────────────────

PLANETS = {
    swe.SUN:     "Sun",
    swe.MOON:    "Moon",
    swe.MERCURY: "Mercury",
    swe.VENUS:   "Venus",
    swe.MARS:    "Mars",
    swe.JUPITER: "Jupiter",
    swe.SATURN:  "Saturn",
    swe.URANUS:  "Uranus",
    swe.NEPTUNE: "Neptune",
    swe.PLUTO:   "Pluto",
    swe.MEAN_NODE: "North Node",  # Rahu
}

SIGNS = [
    "Aries", "Taurus", "Gemini", "Cancer", "Leo", "Virgo",
    "Libra", "Scorpio", "Sagittarius", "Capricorn", "Aquarius", "Pisces"
]

SIGNS_VEDIC = [
    "Mesha", "Vrishabha", "Mithuna", "Karka", "Simha", "Kanya",
    "Tula", "Vrishchika", "Dhanu", "Makara", "Kumbha", "Meena"
]

NAKSHATRAS = [
    ("Ashwini",      "Ketu",    "Horse head"),
    ("Bharani",      "Venus",   "Yoni"),
    ("Krittika",     "Sun",     "Razor/flame"),
    ("Rohini",       "Moon",    "Chariot"),
    ("Mrigashira",   "Mars",    "Deer head"),
    ("Ardra",        "Rahu",    "Teardrop"),
    ("Punarvasu",    "Jupiter", "Quiver of arrows"),
    ("Pushya",       "Saturn",  "Flower/udder"),
    ("Ashlesha",     "Mercury", "Serpent"),
    ("Magha",        "Ketu",    "Throne"),
    ("Purva Phalguni","Venus",  "Hammock"),
    ("Uttara Phalguni","Sun",   "Fig tree"),
    ("Hasta",        "Moon",    "Hand"),
    ("Chitra",       "Mars",    "Pearl"),
    ("Swati",        "Rahu",    "Coral/sword"),
    ("Vishakha",     "Jupiter", "Triumphal arch"),
    ("Anuradha",     "Saturn",  "Lotus"),
    ("Jyeshtha",     "Mercury", "Circular amulet"),
    ("Mula",         "Ketu",    "Bunch of roots"),
    ("Purva Ashadha","Venus",   "Elephant tusk"),
    ("Uttara Ashadha","Sun",    "Elephant tusk"),
    ("Shravana",     "Moon",    "Ear/three footprints"),
    ("Dhanishtha",   "Mars",    "Drum"),
    ("Shatabhisha",  "Rahu",    "Empty circle"),
    ("Purva Bhadrapada","Jupiter","Sword/two front legs of funeral cot"),
    ("Uttara Bhadrapada","Saturn","Two rear legs of funeral cot"),
    ("Revati",       "Mercury", "Fish/drum"),
]

DASHA_SEQUENCE = [
    ("Ketu",    7),
    ("Venus",   20),
    ("Sun",     6),
    ("Moon",    10),
    ("Mars",    7),
    ("Rahu",    18),
    ("Jupiter", 16),
    ("Saturn",  19),
    ("Mercury", 17),
]

LAHIRI_AYANAMSA_1994 = 23.666  # degrees, approximate for 1994

# ── Geocoding ──────────────────────────────────────────────────────────────────

def get_coordinates(place_name):
    """Convert place name to lat/lon/timezone."""
    geolocator = Nominatim(user_agent="astro-synthesis")
    location = geolocator.geocode(place_name)
    if not location:
        raise ValueError(f"Could not find coordinates for: {place_name}")
    
    tf = TimezoneFinder()
    tz_name = tf.timezone_at(lat=location.latitude, lng=location.longitude)
    
    return location.latitude, location.longitude, tz_name, location.address

# ── Julian Day conversion ──────────────────────────────────────────────────────

def to_julian_day(date_str, time_str, tz_name):
    """Convert local datetime to Julian Day (UT) for Swiss Ephemeris."""
    tz = pytz.timezone(tz_name)
    local_dt = datetime.strptime(f"{date_str} {time_str}", "%Y-%m-%d %H:%M")
    local_dt = tz.localize(local_dt)
    utc_dt = local_dt.astimezone(pytz.utc)
    
    jd = swe.julday(
        utc_dt.year, utc_dt.month, utc_dt.day,
        utc_dt.hour + utc_dt.minute / 60.0 + utc_dt.second / 3600.0
    )
    return jd, utc_dt

# ── Degree helpers ─────────────────────────────────────────────────────────────

def deg_to_sign(longitude):
    """Convert ecliptic longitude to sign + degree."""
    sign_idx = int(longitude / 30)
    degree = longitude % 30
    return sign_idx, degree

def format_deg(degree):
    """Format degree as D°M'S\"."""
    d = int(degree)
    m = int((degree - d) * 60)
    s = int(((degree - d) * 60 - m) * 60)
    return f"{d}°{m}'{s}\""

def get_nakshatra(sidereal_longitude):
    """Get nakshatra, pada, and remaining balance from sidereal longitude."""
    nak_idx = int(sidereal_longitude / (360 / 27))
    nak_longitude = sidereal_longitude % (360 / 27)
    pada = int(nak_longitude / (360 / 27 / 4)) + 1
    
    # Balance remaining in nakshatra (for dasha calculation)
    nak_span = 360 / 27  # 13.333... degrees
    balance_fraction = 1 - (nak_longitude / nak_span)
    
    nak = NAKSHATRAS[nak_idx]
    return nak_idx, nak[0], nak[1], nak[2], pada, balance_fraction

# ── Planet calculation ─────────────────────────────────────────────────────────

def calculate_planets(jd, ayanamsa=0):
    """Calculate all planetary positions. ayanamsa=0 for Western, Lahiri value for Vedic."""
    positions = {}
    
    for planet_id, planet_name in PLANETS.items():
        try:
            result, _ = swe.calc_ut(jd, planet_id)
            tropical_lon = result[0]
            sidereal_lon = (tropical_lon - ayanamsa) % 360
            speed = result[3]  # degrees/day — negative = retrograde
            
            w_sign_idx, w_deg = deg_to_sign(tropical_lon)
            v_sign_idx, v_deg = deg_to_sign(sidereal_lon)
            
            nak_idx, nak_name, nak_ruler, nak_symbol, pada, nak_balance = get_nakshatra(sidereal_lon)
            
            positions[planet_name] = {
                "tropical_longitude": round(tropical_lon, 4),
                "sidereal_longitude": round(sidereal_lon, 4),
                "western": {
                    "sign": SIGNS[w_sign_idx],
                    "degree": format_deg(w_deg),
                    "degree_raw": round(w_deg, 4),
                },
                "vedic": {
                    "sign": SIGNS_VEDIC[v_sign_idx],
                    "degree": format_deg(v_deg),
                    "degree_raw": round(v_deg, 4),
                    "nakshatra": nak_name,
                    "nakshatra_ruler": nak_ruler,
                    "nakshatra_symbol": nak_symbol,
                    "pada": pada,
                    "nakshatra_balance": round(nak_balance, 4),
                },
                "retrograde": speed < 0,
                "speed_deg_day": round(speed, 4),
            }
        except Exception as e:
            positions[planet_name] = {"error": str(e)}
    
    # South Node (Ketu) = North Node + 180
    if "North Node" in positions and "error" not in positions["North Node"]:
        rahu_trop = positions["North Node"]["tropical_longitude"]
        ketu_trop = (rahu_trop + 180) % 360
        ketu_sid = (ketu_trop - ayanamsa) % 360
        
        w_sign_idx, w_deg = deg_to_sign(ketu_trop)
        v_sign_idx, v_deg = deg_to_sign(ketu_sid)
        nak_idx, nak_name, nak_ruler, nak_symbol, pada, nak_balance = get_nakshatra(ketu_sid)
        
        positions["South Node (Ketu)"] = {
            "tropical_longitude": round(ketu_trop, 4),
            "sidereal_longitude": round(ketu_sid, 4),
            "western": {
                "sign": SIGNS[w_sign_idx],
                "degree": format_deg(w_deg),
                "degree_raw": round(w_deg, 4),
            },
            "vedic": {
                "sign": SIGNS_VEDIC[v_sign_idx],
                "degree": format_deg(v_deg),
                "degree_raw": round(v_deg, 4),
                "nakshatra": nak_name,
                "nakshatra_ruler": nak_ruler,
                "nakshatra_symbol": nak_symbol,
                "pada": pada,
            },
            "retrograde": True,  # Ketu always retrograde
        }
    
    return positions

# ── Ascendant / Houses ─────────────────────────────────────────────────────────

def calculate_houses(jd, lat, lon, ayanamsa=0, house_system=b'P'):
    """
    Calculate houses. 
    house_system: b'P' = Placidus (Western default), b'W' = Whole Sign (Vedic default)
    """
    cusps, ascmc = swe.houses(jd, lat, lon, house_system)
    
    asc_tropical = ascmc[0]
    mc_tropical = ascmc[1]
    
    asc_sidereal = (asc_tropical - ayanamsa) % 360
    mc_sidereal = (mc_tropical - ayanamsa) % 360
    
    asc_w_sign, asc_w_deg = deg_to_sign(asc_tropical)
    asc_v_sign, asc_v_deg = deg_to_sign(asc_sidereal)
    mc_w_sign, mc_w_deg = deg_to_sign(mc_tropical)
    mc_v_sign, mc_v_deg = deg_to_sign(mc_sidereal)
    
    nak_idx, nak_name, nak_ruler, nak_symbol, pada, nak_balance = get_nakshatra(asc_sidereal)
    
    houses = {
        "ascendant": {
            "western": {
                "sign": SIGNS[asc_w_sign],
                "degree": format_deg(asc_w_deg),
            },
            "vedic": {
                "sign": SIGNS_VEDIC[asc_v_sign],
                "degree": format_deg(asc_v_deg),
                "nakshatra": nak_name,
                "nakshatra_ruler": nak_ruler,
                "pada": pada,
            }
        },
        "midheaven_mc": {
            "western": {
                "sign": SIGNS[mc_w_sign],
                "degree": format_deg(mc_w_deg),
            },
            "vedic": {
                "sign": SIGNS_VEDIC[mc_v_sign],
                "degree": format_deg(mc_v_deg),
            }
        },
        "house_cusps_western": []
    }
    
    for i, cusp in enumerate(cusps):
        sign_idx, deg = deg_to_sign(cusp)
        houses["house_cusps_western"].append({
            "house": i + 1,
            "sign": SIGNS[sign_idx],
            "degree": format_deg(deg),
            "longitude": round(cusp, 4),
        })
    
    return houses

# ── Dasha calculation ──────────────────────────────────────────────────────────

def calculate_dasha(birth_date_str, moon_nakshatra_ruler, moon_nak_balance):
    """Calculate Vimshottari Dasha from birth."""
    
    # Find which dasha the Moon nakshatra ruler belongs to
    dasha_start_idx = next(
        i for i, (planet, _) in enumerate(DASHA_SEQUENCE) 
        if planet == moon_nakshatra_ruler
    )
    
    birth_date = datetime.strptime(birth_date_str, "%Y-%m-%d")
    
    # First dasha: only the remaining balance
    first_planet, first_years = DASHA_SEQUENCE[dasha_start_idx]
    first_duration_years = first_years * moon_nak_balance
    
    dashas = []
    current_date = birth_date
    
    # First partial dasha
    end_date = datetime(
        birth_date.year + int(first_duration_years),
        birth_date.month,
        birth_date.day
    )
    dashas.append({
        "planet": first_planet,
        "years": round(first_duration_years, 2),
        "start": birth_date.strftime("%Y-%m-%d"),
        "end": end_date.strftime("%Y-%m-%d"),
        "partial": True
    })
    current_date = end_date
    
    # Remaining full dashas (enough for 120 years)
    for i in range(1, 27):
        idx = (dasha_start_idx + i) % 9
        planet, years = DASHA_SEQUENCE[idx]
        end_year = current_date.year + years
        try:
            end_date = datetime(end_year, current_date.month, current_date.day)
        except ValueError:
            end_date = datetime(end_year, current_date.month, 28)
        
        dashas.append({
            "planet": planet,
            "years": years,
            "start": current_date.strftime("%Y-%m-%d"),
            "end": end_date.strftime("%Y-%m-%d"),
            "partial": False
        })
        current_date = end_date
        
        if current_date.year > birth_date.year + 120:
            break
    
    # Find current dasha
    today = datetime.now()
    current_dasha = next(
        (d for d in dashas if 
         datetime.strptime(d["start"], "%Y-%m-%d") <= today <= datetime.strptime(d["end"], "%Y-%m-%d")),
        None
    )
    
    return dashas, current_dasha

# ── BaZi calculation ───────────────────────────────────────────────────────────

HEAVENLY_STEMS = ["甲","乙","丙","丁","戊","己","庚","辛","壬","癸"]
EARTHLY_BRANCHES = ["子","丑","寅","卯","辰","巳","午","未","申","酉","戌","亥"]
STEM_NAMES = ["Jiǎ (Yang Wood)","Yǐ (Yin Wood)","Bǐng (Yang Fire)","Dīng (Yin Fire)",
              "Wù (Yang Earth)","Jǐ (Yin Earth)","Gēng (Yang Metal)","Xīn (Yin Metal)",
              "Rén (Yang Water)","Guǐ (Yin Water)"]
BRANCH_NAMES = ["Rat (Water)","Ox (Earth)","Tiger (Wood)","Rabbit (Wood)",
                "Dragon (Earth)","Snake (Fire)","Horse (Fire)","Goat (Earth)",
                "Monkey (Metal)","Rooster (Metal)","Dog (Earth)","Pig (Water)"]
BRANCH_ANIMALS = ["Rat","Ox","Tiger","Rabbit","Dragon","Snake",
                  "Horse","Goat","Monkey","Rooster","Dog","Pig"]

def get_bazi_year_pillar(year):
    stem_idx = (year - 4) % 10
    branch_idx = (year - 4) % 12
    return HEAVENLY_STEMS[stem_idx], EARTHLY_BRANCHES[branch_idx], STEM_NAMES[stem_idx], BRANCH_NAMES[branch_idx]

def get_bazi_month_pillar(year, month, day):
    """Get BaZi month pillar based on solar terms."""
    # Solar term start dates (approximate) for each month pillar
    # Month pillar changes at solar terms, not calendar months
    solar_term_days = [6,4,6,5,6,6,7,7,8,8,7,7]  # approximate start days
    
    # Determine which solar month we're in
    if day >= solar_term_days[month - 1]:
        solar_month = month
    else:
        solar_month = month - 1 if month > 1 else 12
        if solar_month == 12:
            year -= 1
    
    year_stem_idx = (year - 4) % 10
    # Branch: Tiger(2)=Feb, Rabbit(3)=Mar...Pig(11)=Nov, Rat(0)=Dec, Ox(1)=Jan
    branch_idx = solar_month % 12
    # Stem: Tiger month stem depends on year stem group
    # 甲/己→丙(2), 乙/庚→戊(4), 丙/辛→庚(6), 丁/壬→壬(8), 戊/癸→甲(0)
    tiger_stem = [2, 4, 6, 8, 0][year_stem_idx % 5]
    stem_idx = (tiger_stem + solar_month - 2) % 10
    
    return HEAVENLY_STEMS[stem_idx], EARTHLY_BRANCHES[branch_idx], STEM_NAMES[stem_idx], BRANCH_NAMES[branch_idx]

def get_bazi_day_pillar(jd):
    """Get BaZi day pillar from Julian Day number."""
    # Day 0 of the 60-cycle was Jan 1, 1900 (known reference: stem=甲, branch=子)
    # JD for Jan 1, 1900 = 2415021.0
    ref_jd = 2415021.0
    days_diff = int(jd - ref_jd)
    
    stem_idx = (days_diff + 0) % 10   # 甲=0 on reference day
    branch_idx = (days_diff + 0) % 12  # 子=0 on reference day
    
    return HEAVENLY_STEMS[stem_idx], EARTHLY_BRANCHES[branch_idx], STEM_NAMES[stem_idx], BRANCH_NAMES[branch_idx]

def get_bazi_hour_pillar(hour_24, day_stem):
    """Get BaZi hour pillar. Hour stem depends on day stem."""
    # Two-hour periods: Rat=23-1, Ox=1-3, Tiger=3-5...
    branch_idx = int((hour_24 + 1) / 2) % 12
    
    day_stem_idx = HEAVENLY_STEMS.index(day_stem)
    hour_stem_base = (day_stem_idx % 5) * 2
    stem_idx = (hour_stem_base + branch_idx) % 10
    
    return HEAVENLY_STEMS[stem_idx], EARTHLY_BRANCHES[branch_idx], STEM_NAMES[stem_idx], BRANCH_NAMES[branch_idx]

def calculate_bazi(date_str, time_str, jd, utc_dt):
    """Calculate full BaZi Four Pillars."""
    dt = datetime.strptime(f"{date_str} {time_str}", "%Y-%m-%d %H:%M")
    
    y_stem, y_branch, y_stem_name, y_branch_name = get_bazi_year_pillar(dt.year)
    m_stem, m_branch, m_stem_name, m_branch_name = get_bazi_month_pillar(dt.year, dt.month, dt.day)
    d_stem, d_branch, d_stem_name, d_branch_name = get_bazi_day_pillar(jd)
    h_stem, h_branch, h_stem_name, h_branch_name = get_bazi_hour_pillar(dt.hour, d_stem)
    
    # Check for clashes (opposite branches)
    clashes = {
        "子": "午", "午": "子", "丑": "未", "未": "丑",
        "寅": "申", "申": "寅", "卯": "酉", "酉": "卯",
        "辰": "戌", "戌": "辰", "巳": "亥", "亥": "巳"
    }
    
    branches = [y_branch, m_branch, d_branch, h_branch]
    pillar_names = ["Year", "Month", "Day", "Hour"]
    
    found_clashes = []
    for i in range(len(branches)):
        for j in range(i + 1, len(branches)):
            if clashes.get(branches[i]) == branches[j]:
                found_clashes.append(f"{pillar_names[i]} {BRANCH_ANIMALS[EARTHLY_BRANCHES.index(branches[i])]} clashes with {pillar_names[j]} {BRANCH_ANIMALS[EARTHLY_BRANCHES.index(branches[j])]}")
    
    return {
        "year_pillar":  {"stem": y_stem, "branch": y_branch, "stem_name": y_stem_name, "branch_name": y_branch_name},
        "month_pillar": {"stem": m_stem, "branch": m_branch, "stem_name": m_stem_name, "branch_name": m_branch_name},
        "day_pillar":   {"stem": d_stem, "branch": d_branch, "stem_name": d_stem_name, "branch_name": d_branch_name, "note": "Day Master — this is YOU"},
        "hour_pillar":  {"stem": h_stem, "branch": h_branch, "stem_name": h_stem_name, "branch_name": h_branch_name},
        "clashes": found_clashes,
        "day_master": f"{d_stem} — {d_stem_name}",
    }

# ── Numerology ─────────────────────────────────────────────────────────────────

def calculate_numerology(date_str, name=None):
    dt = datetime.strptime(date_str, "%Y-%m-%d")
    
    def reduce(n):
        while n > 9 and n not in (11, 22, 33):
            n = sum(int(d) for d in str(n))
        return n
    
    day = reduce(dt.day)
    month = reduce(dt.month)
    year = reduce(sum(int(d) for d in str(dt.year)))
    life_path = reduce(day + month + year)
    
    birth_day = reduce(dt.day)
    
    # Personal year (current)
    current_year = datetime.now().year
    personal_year = reduce(
        reduce(dt.day) + reduce(dt.month) + reduce(sum(int(d) for d in str(current_year)))
    )
    
    result = {
        "life_path": life_path,
        "birth_day_number": birth_day,
        "personal_year_current": personal_year,
        "personal_year_next": reduce(
            reduce(dt.day) + reduce(dt.month) + reduce(sum(int(d) for d in str(current_year + 1)))
        ),
        "calculation_breakdown": {
            "day": f"{dt.day} → {day}",
            "month": f"{dt.month} → {month}",
            "year": f"{dt.year} → {year}",
            "life_path": f"{day} + {month} + {year} = {day+month+year} → {life_path}"
        }
    }
    
    if name:
        pythagorean = {
            'a':1,'b':2,'c':3,'d':4,'e':5,'f':6,'g':7,'h':8,'i':9,
            'j':1,'k':2,'l':3,'m':4,'n':5,'o':6,'p':7,'q':8,'r':9,
            's':1,'t':2,'u':3,'v':4,'w':5,'x':6,'y':7,'z':8
        }
        vowels = set('aeiou')
        name_clean = name.lower().replace(" ", "")
        
        expression = reduce(sum(pythagorean.get(c, 0) for c in name_clean))
        soul_urge = reduce(sum(pythagorean.get(c, 0) for c in name_clean if c in vowels))
        personality = reduce(sum(pythagorean.get(c, 0) for c in name_clean if c not in vowels))
        
        result["expression_destiny"] = expression
        result["soul_urge"] = soul_urge
        result["personality_number"] = personality
    
    return result

# ── Main ───────────────────────────────────────────────────────────────────────

def calculate_full_chart(date_str, time_str, lat=None, lon=None, place=None, tz_name=None, name=None):
    """Master function — returns complete chart data."""
    
    # Get coordinates if place name given
    if place and (lat is None or lon is None):
        print(f"📍 Geocoding: {place}...")
        lat, lon, tz_name, full_address = get_coordinates(place)
        print(f"   Found: {full_address}")
        print(f"   Coordinates: {lat:.4f}°N, {lon:.4f}°E")
        print(f"   Timezone: {tz_name}")
    
    if tz_name is None:
        tf = TimezoneFinder()
        tz_name = tf.timezone_at(lat=lat, lng=lon)
    
    print(f"\n⚙️  Calculating chart for {date_str} {time_str} {tz_name}...")
    
    # Julian Day
    jd, utc_dt = to_julian_day(date_str, time_str, tz_name)
    
    # Ayanamsa (Lahiri — standard for Vedic)
    ayanamsa = swe.get_ayanamsa_ut(jd)
    
    # Calculate everything
    planets = calculate_planets(jd, ayanamsa)
    houses_western = calculate_houses(jd, lat, lon, ayanamsa=0, house_system=b'P')  # Placidus
    houses_vedic = calculate_houses(jd, lat, lon, ayanamsa=ayanamsa, house_system=b'W')  # Whole sign
    bazi = calculate_bazi(date_str, time_str, jd, utc_dt)
    numerology = calculate_numerology(date_str, name)
    
    # Dasha from Moon nakshatra
    moon_data = planets.get("Moon", {})
    moon_nak_ruler = moon_data.get("vedic", {}).get("nakshatra_ruler", "Moon")
    moon_nak_balance = moon_data.get("vedic", {}).get("nakshatra_balance", 0.5)
    dashas, current_dasha = calculate_dasha(date_str, moon_nak_ruler, moon_nak_balance)
    
    chart = {
        "input": {
            "date": date_str,
            "time": time_str,
            "latitude": round(lat, 4),
            "longitude": round(lon, 4),
            "timezone": tz_name,
            "name": name,
        },
        "ayanamsa_lahiri": round(ayanamsa, 4),
        "planets": planets,
        "western": {
            "houses": houses_western,
            "note": "Placidus house system"
        },
        "vedic": {
            "houses": houses_vedic,
            "lagna": houses_vedic["ascendant"]["vedic"],
            "note": "Whole sign house system, Lahiri ayanamsa"
        },
        "dasha": {
            "current": current_dasha,
            "sequence": dashas[:12],  # Next 12 periods
        },
        "bazi": bazi,
        "numerology": numerology,
    }
    
    return chart

def print_chart_summary(chart):
    """Pretty print the chart for terminal reading."""
    p = chart["planets"]
    w = chart["western"]["houses"]
    v = chart["vedic"]
    d = chart["dasha"]
    b = chart["bazi"]
    n = chart["numerology"]
    
    print("\n" + "═"*60)
    print("  🔮 ASTRO-SYNTHESIS CHART")
    print("═"*60)
    
    print(f"\n📅 Birth: {chart['input']['date']} {chart['input']['time']}")
    print(f"📍 Location: {chart['input']['latitude']}°N, {chart['input']['longitude']}°E")
    print(f"🔭 Ayanamsa (Lahiri): {chart['ayanamsa_lahiri']}°")
    
    print("\n── WESTERN CHART ──────────────────────────────────────")
    print(f"  Ascendant:  {w['ascendant']['western']['sign']} {w['ascendant']['western']['degree']}")
    print(f"  Midheaven:  {w['midheaven_mc']['western']['sign']} {w['midheaven_mc']['western']['degree']}")
    
    for planet, data in p.items():
        if "error" not in data:
            retro = " ℞" if data.get("retrograde") else ""
            print(f"  {planet:<18} {data['western']['sign']:<14} {data['western']['degree']}{retro}")
    
    print("\n── VEDIC / JYOTISH ────────────────────────────────────")
    print(f"  Lagna:      {v['lagna']['sign']} {v['lagna']['degree']} | {v['lagna']['nakshatra']} pada {v['lagna']['pada']}")
    
    for planet, data in p.items():
        if "error" not in data:
            retro = " ℞" if data.get("retrograde") else ""
            vd = data["vedic"]
            print(f"  {planet:<18} {vd['sign']:<14} {vd['degree']} | {vd['nakshatra']} (ruler: {vd['nakshatra_ruler']}){retro}")
    
    print("\n── VIMSHOTTARI DASHA ──────────────────────────────────")
    if d["current"]:
        cd = d["current"]
        print(f"  Current:    {cd['planet']} Mahadasha  ({cd['start']} → {cd['end']})")
    for dasha in d["sequence"][:6]:
        marker = " ◀ NOW" if dasha == d["current"] else ""
        print(f"  {dasha['planet']:<12} {dasha['years']} yrs  {dasha['start']} → {dasha['end']}{marker}")
    
    print("\n── BAZI (四柱命理) ────────────────────────────────────")
    for pillar in ["year_pillar", "month_pillar", "day_pillar", "hour_pillar"]:
        pd = b[pillar]
        label = pillar.replace("_pillar","").capitalize()
        note = " ← YOU (Day Master)" if pillar == "day_pillar" else ""
        print(f"  {label:<8} {pd['stem']}{pd['branch']}  {pd['stem_name']} / {pd['branch_name']}{note}")
    
    if b["clashes"]:
        print(f"\n  ⚡ Clashes: {', '.join(b['clashes'])}")
    
    print("\n── NUMEROLOGY ─────────────────────────────────────────")
    print(f"  Life Path:       {n['life_path']}")
    print(f"  Birth Day:       {n['birth_day_number']}")
    print(f"  Personal Year ({chart['input']['date'][:4][:3]}now):  {n['personal_year_current']}")
    print(f"  Personal Year (next): {n['personal_year_next']}")
    if "expression_destiny" in n:
        print(f"  Expression:      {n['expression_destiny']}")
        print(f"  Soul Urge:       {n['soul_urge']}")
    
    print("\n" + "═"*60)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Astro-Synthesis Chart Calculator")
    parser.add_argument("--date",  required=True, help="Birth date YYYY-MM-DD")
    parser.add_argument("--time",  required=True, help="Birth time HH:MM (24hr)")
    parser.add_argument("--place", help="Birth place (city, country)")
    parser.add_argument("--lat",   type=float, help="Latitude (if not using --place)")
    parser.add_argument("--lon",   type=float, help="Longitude (if not using --place)")
    parser.add_argument("--tz",    help="Timezone (e.g. Asia/Kolkata)")
    parser.add_argument("--name",  help="Full birth name (for numerology expression number)")
    parser.add_argument("--json",  action="store_true", help="Output raw JSON instead of summary")
    
    args = parser.parse_args()
    
    chart = calculate_full_chart(
        date_str=args.date,
        time_str=args.time,
        lat=args.lat,
        lon=args.lon,
        place=args.place,
        tz_name=args.tz,
        name=args.name,
    )
    
    if args.json:
        print(json.dumps(chart, indent=2, ensure_ascii=False))
    else:
        print_chart_summary(chart)
        print("\n💾 Full JSON available with --json flag")
