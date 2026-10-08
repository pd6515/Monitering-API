"""Load demo master data and one day of hourly logs:  python seed.py [YYYY-MM-DD]"""
import random, sys
from datetime import date
from sqlalchemy import select
from app.database import Base, engine, SessionLocal
from app import models as m, schemas as s, services as svc

day = date.fromisoformat(sys.argv[1]) if len(sys.argv) > 1 else date.today()
Base.metadata.create_all(engine)
random.seed(7)
with SessionLocal() as db:
    def ensure(model, code, name, **kw):
        if not db.scalar(select(model).where(model.code == code)):
            db.add(model(code=code, name=name, **kw))
    for c, n in [("V01", "Alpha Components"), ("V02", "Beta Plastics"), ("V03", "Gamma Packaging")]:
        ensure(m.Vendor, c, n)
    for c, n in [("P100", "Speaker driver"), ("P200", "Enclosure"), ("P300", "Retail box")]:
        ensure(m.Part, c, n)
    for c, n in [("CUT", "Cutting"), ("WLD", "Welding"), ("ASM", "Assembly")]:
        ensure(m.Process, c, n)
    db.commit()

    combos = [("V01", "P100", "ASM"), ("V02", "P200", "CUT"), ("V03", "P300", "ASM")]
    entries = []
    for hour in range(8, 17):                       # 08:00 - 17:00 shift
        for v, p, pr in combos:
            plan = 100
            prod = max(0, plan - random.randint(0, 30))
            entries.append(s.HourlyLogIn(log_date=day, hour=hour, vendor_code=v, part_code=p, process_code=pr,
                                         planned_qty=plan, produced_qty=prod, rejected_qty=random.randint(0, 4),
                                         downtime_min=random.choice([0, 0, 0, 5, 10])))
    created, updated = svc.upsert_logs(db, entries)
    print(f"Seeded {day}: {created} created, {updated} updated")
