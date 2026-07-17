# 09 — Database Integration with Flask (SQLAlchemy)

Recap file 08's closing note and the entire SQL folder directly — connecting a real database to persist data beyond what a stateless API alone can hold, using SQLAlchemy's Python ORM syntax instead of raw SQL queries.

## What an ORM actually is — recap SQL file 05's joins and file 09's schema design

```python
# An ORM (Object-Relational Mapper) lets you interact with a database using
# PYTHON CLASSES and OBJECTS, instead of writing raw SQL strings directly
# Genuinely just a translation layer -- every ORM operation eventually becomes
# real SQL under the hood (recap the ENTIRE SQL folder -- that knowledge doesn't
# become obsolete, it's precisely what's happening beneath the ORM syntax)
```

## Setting up Flask-SQLAlchemy

```python
from flask_sqlalchemy import SQLAlchemy

app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'      # recap SQL file 01's SQLite setup directly
db = SQLAlchemy(app)
```

## Defining a model — recap SQL file 09's schema/constraints directly

```python
class Prediction(db.Model):
    id = db.Column(db.Integer, primary_key=True)              # recap SQL file 09's PRIMARY KEY
    input_text = db.Column(db.Text, nullable=False)              # recap SQL file 09's NOT NULL constraint
    predicted_label = db.Column(db.String(50), nullable=False)
    confidence = db.Column(db.Float)
    created_at = db.Column(db.DateTime, server_default=db.func.now())      # recap SQL file 09's DEFAULT

    def __repr__(self):
        return f"<Prediction {self.id}: {self.predicted_label}>"
```

Genuinely worth recognizing this as EXACTLY SQL file 09's `CREATE TABLE` syntax, just expressed as a Python class instead of raw SQL — `db.Column(db.Integer, primary_key=True)` genuinely generates the same `id INTEGER PRIMARY KEY` from that file, `nullable=False` generates the same `NOT NULL` constraint.

## Creating the actual database tables

```python
with app.app_context():
    db.create_all()      # genuinely creates the actual tables based on every db.Model subclass defined
```

## CRUD operations — recap SQL files 02 and 10 directly, now via ORM syntax

```python
# CREATE -- recap SQL file 10's INSERT
new_prediction = Prediction(input_text="Great movie!", predicted_label="positive", confidence=0.95)
db.session.add(new_prediction)
db.session.commit()      # recap SQL file 10's transaction/COMMIT discipline directly

# READ -- recap SQL file 02's SELECT/WHERE
all_predictions = Prediction.query.all()
positive_predictions = Prediction.query.filter_by(predicted_label='positive').all()
recent = Prediction.query.order_by(Prediction.created_at.desc()).limit(10).all()      # recap SQL file 03

# UPDATE -- recap SQL file 10's UPDATE
prediction = Prediction.query.get(1)
prediction.confidence = 0.99
db.session.commit()

# DELETE -- recap SQL file 10's DELETE, and its "verify with SELECT first" safety discipline
prediction_to_delete = Prediction.query.get(1)
db.session.delete(prediction_to_delete)
db.session.commit()
```

Genuinely worth pausing on how directly this maps — `.query.filter_by()` is `WHERE`, `.order_by().limit()` is `ORDER BY ... LIMIT` (recap SQL file 03's exact syntax parallel), `db.session.commit()` is genuinely the SAME transaction commit concept from SQL file 10, just called through Python instead of a raw `COMMIT;` statement.

## Logging every prediction — a genuinely realistic, practical use case

```python
@api_bp.route('/predict', methods=['POST'])
def predict():
    data = request.get_json()
    text = data.get('text')

    result = sentiment_model.predict(text)      # recap file 08's model class

    # Genuinely realistic practice: log every prediction for later analysis
    log_entry = Prediction(
        input_text=text,
        predicted_label=result['prediction'],
        confidence=result['confidence']
    )
    db.session.add(log_entry)
    db.session.commit()

    return jsonify(result), 200
```

Genuinely a real, valuable pattern connecting directly to the general ML-monitoring themes that'll matter more in the upcoming MLOps folder — logging every prediction to a database enables LATER analysis: tracking prediction volume over time, spotting drift in input patterns, building a dataset of real usage for potential future retraining.

## Relationships between tables — recap SQL file 05's foreign keys and joins directly

```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    predictions = db.relationship('Prediction', backref='user', lazy=True)      # ONE-to-MANY

class Prediction(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    input_text = db.Column(db.Text, nullable=False)
    predicted_label = db.Column(db.String(50))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)      # recap SQL file 01/09's FOREIGN KEY
```

```python
user = User.query.get(1)
print(user.predictions)      # SQLAlchemy automatically handles the JOIN (recap SQL file 05) behind the scenes

prediction = Prediction.query.get(1)
print(prediction.user.username)      # `backref` makes this reverse lookup possible too
```

Genuinely satisfying to see — `db.relationship` and `db.ForeignKey` are the ORM's way of expressing EXACTLY the primary-key/foreign-key relationship from SQL file 01/09/05, and accessing `user.predictions` genuinely triggers a real `JOIN` query under the hood, even though it looks like simple Python attribute access.

## Aggregation queries — recap SQL file 04's GROUP BY directly

```python
from sqlalchemy import func

label_counts = db.session.query(
    Prediction.predicted_label, func.count(Prediction.id)
).group_by(Prediction.predicted_label).all()

print(label_counts)   # [('positive', 45), ('negative', 32)]
```

Genuinely worth recognizing `func.count()` and `.group_by()` as the DIRECT ORM equivalent of SQL file 04's `COUNT(*)` and `GROUP BY` — the SAME aggregation concept, same underlying SQL genuinely being generated and executed.

## Raw SQL when the ORM genuinely isn't enough

```python
result = db.session.execute(db.text("SELECT predicted_label, AVG(confidence) FROM prediction GROUP BY predicted_label"))
```

Genuinely worth knowing this escape hatch exists — for GENUINELY complex queries (recap SQL file 07's window functions, or intricate CTEs from file 06), sometimes writing raw SQL directly is clearer and more efficient than fighting the ORM's abstraction. The entire SQL folder's knowledge remains directly usable, not replaced by learning the ORM.

## Database migrations — a genuinely important, easy-to-overlook production concern

```python
# Genuinely important: once a real app is running with real data, changing a
# model's schema (adding a column, renaming a field) can't just be done by
# editing the Python class -- the EXISTING database needs to be migrated
# Flask-Migrate (built on Alembic) handles this properly:
# flask db init / flask db migrate / flask db upgrade
```

Genuinely worth knowing this exists as the standard, correct tool for evolving a database schema over time without losing existing data — directly relevant once file 11's deployment makes the database a genuinely persistent, real production concern rather than a disposable local file.

## Try this

```python
# Extend the Flask app from file 08's exercise with a Prediction model that logs
# every prediction request (input text, predicted label, confidence, timestamp)
# Add a new endpoint GET /api/v1/history that returns the 10 most recent
# predictions as JSON, using .order_by().limit() (recap SQL file 03 directly)
# Add another endpoint GET /api/v1/stats that returns aggregate counts per
# predicted label using func.count() and .group_by() (recap SQL file 04 directly)
# Make several prediction requests, then confirm both new endpoints correctly
# reflect the logged history and aggregated statistics
```

Building both the history and stats endpoints is genuinely a good, complete test of the CRUD + aggregation skills this file covers, and directly produces a genuinely useful feature (usage analytics) for a real ML application.

## What's next
File 10 — Authentication & Sessions, adding user accounts and access control — genuinely necessary once an app needs to distinguish between different users or protect certain endpoints from public access.
