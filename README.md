# system-programming-code
UNIT_PRICE = 10   # approved price: UGX per litre

def validate_transaction_id(tid, store):
    tid = str(tid).strip().upper()
    if not ID_PATTERN.match(tid):
        raise ValueError(f"invalid transaction ID '{tid}' (expected e.g. W001)")
    if tid in store:
        raise ValueError(f"duplicate transaction ID '{tid}'")
    return tid

def validate_quantity(quantity):
    text = str(quantity).strip()
    if text == "":
        raise ValueError("quantity is missing")
    try:
        value = int(text)
    except ValueError:
        raise ValueError(f"quantity '{text}' is not a whole number")
    if value < 0:
        raise ValueError(f"quantity {value} is negative")
    if value == 0:
        raise ValueError("zero-litre transaction is not a valid sale")
    return value

def calculate_payment(quantity):
    return quantity * UNIT_PRICE

def register_transaction(store, tid, customer, point, quantity):
    try:
        tid = validate_transaction_id(tid, store)
        customer = validate_customer(customer)
        point = validate_supply_point(point)
        quantity = validate_quantity(quantity)
    except ValueError as err:
        return False, str(err)
    record = {"id": tid, "customer": customer, "point": point,
              "quantity": quantity, "amount": calculate_payment(quantity)}
    store[tid] = record
    return True, record

def search_by_customer(store, customer):
    customer = str(customer).strip().upper()
    return [r for r in store.values() if r["customer"] == customer]

def total_quantity(store):
    return sum(r["quantity"] for r in store.values())

def total_revenue(store):
    return sum(r["amount"] for r in store.values())

def quantity_by_customer(store):
    totals = {}
    for r in store.values():
        totals[r["customer"]] = totals.get(r["customer"], 0) + r["quantity"]
    return totals

def top_customer(store):
    totals = quantity_by_customer(store)
    if not totals:
        return None, 0
    best = max(totals, key=totals.get)
    return best, totals[best]
