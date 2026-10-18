# Vectox-1.1
VICTOR — Vector, Intelligence, Calculations, Trajectory, Orientation & Robotics
A beginner-friendly physics and mathematics assistant.

Requirements: Python 3.10+ (standard library only)
Run: python victor.py

This is a calculation-based assistant, not a trained language model. It uses
validated numerical tools and a menu. All robotics movement is simulated;
it does not control real hardware.
"""

import math
import statistics


APP_NAME = "VICTOR"
EPSILON = 1e-12


def number(prompt):
    """Read a finite number from the user."""
    while True:
        try:
            value = float(input(prompt))
            if not math.isfinite(value):
                raise ValueError
            return value
        except ValueError:
            print("Please enter a valid finite number.")


def positive_number(prompt, allow_zero=True):
    while True:
        value = number(prompt)
        if value > 0 or (allow_zero and value == 0):
            return value
        print("Please enter a value greater than " + ("or equal to " if allow_zero else "") + "zero.")


def vector_input(label="vector"):
    print(f"Enter the x, y, z components of the {label}:")
    return [number("  x = "), number("  y = "), number("  z = ")]


def vector_add(a, b):
    return [x + y for x, y in zip(a, b)]


def vector_subtract(a, b):
    return [x - y for x, y in zip(a, b)]


def vector_scale(a, scalar):
    return [scalar * x for x in a]


def dot(a, b):
    return sum(x * y for x, y in zip(a, b))


def cross(a, b):
    return [
        a[1] * b[2] - a[2] * b[1],
        a[2] * b[0] - a[0] * b[2],
        a[0] * b[1] - a[1] * b[0],
    ]


def magnitude(a):
    return math.sqrt(dot(a, a))


def unit_vector(a):
    mag = magnitude(a)
    if mag < EPSILON:
        raise ValueError("The zero vector has no defined direction.")
    return [x / mag for x in a]


def fmt_vector(v):
    return "(" + ", ".join(f"{x:.6g}" for x in v) + ")"


def angle_between(a, b):
    ma, mb = magnitude(a), magnitude(b)
    if ma < EPSILON or mb < EPSILON:
        raise ValueError("Angle is undefined when either vector is zero.")
    cosine = max(-1.0, min(1.0, dot(a, b) / (ma * mb)))
    return math.degrees(math.acos(cosine))


def vector_tool():
    print("\n--- VECTOR & DIRECTION TOOL ---")
    a = vector_input("vector A")
    print(f"|A| = {magnitude(a):.6g}")
    if magnitude(a) > EPSILON:
        print(f"Unit vector of A = {fmt_vector(unit_vector(a))}")
    print("\nChoose: 1 Add  2 Subtract A-B  3 Dot product  4 Cross product")
    print("        5 Angle between vectors  6 Resolve A along a direction")
    choice = input("Choice: ").strip()
    if choice in {"1", "2", "3", "4", "5", "6"}:
        b = vector_input("vector B / direction")
        try:
            if choice == "1":
                print("A + B =", fmt_vector(vector_add(a, b)))
            elif choice == "2":
                print("A - B =", fmt_vector(vector_subtract(a, b)))
            elif choice == "3":
                print("A · B =", f"{dot(a, b):.6g}")
            elif choice == "4":
                print("A × B =", fmt_vector(cross(a, b)))
            elif choice == "5":
                print("Angle =", f"{angle_between(a, b):.6g} degrees")
            elif choice == "6":
                direction = unit_vector(b)
                scalar_component = dot(a, direction)
                vector_component = vector_scale(direction, scalar_component)
                print("Scalar component of A along B =", f"{scalar_component:.6g}")
                print("Vector projection of A on B =", fmt_vector(vector_component))
        except ValueError as exc:
            print("Cannot calculate:", exc)


def target_direction_tool():
    print("\n--- TARGET DIRECTION / ROBOT ORIENTATION ---")
    current = vector_input("current position (x, y, z)")
    target = vector_input("target position (x, y, z)")
    displacement = vector_subtract(target, current)
    distance = magnitude(displacement)
    print("Displacement to target =", fmt_vector(displacement))
    print(f"Straight-line distance = {distance:.6g} units")
    if distance < EPSILON:
        print("The object is already at the target.")
        return
    direction = unit_vector(displacement)
    print("Required unit direction =", fmt_vector(direction))
    horizontal = math.hypot(displacement[0], displacement[1])
    yaw = math.degrees(math.atan2(displacement[1], displacement[0]))
    pitch = math.degrees(math.atan2(displacement[2], horizontal))
    print(f"Yaw/heading in XY plane = {yaw:.6g} degrees from +X toward +Y")
    print(f"Pitch/elevation = {pitch:.6g} degrees above XY plane")
    speed = positive_number("Desired travel speed (units/s): ", allow_zero=False)
    print(f"Ideal travel time at constant speed = {distance / speed:.6g} s")
    print("Note: this is a geometric suggestion, not a collision-safe robot path.")


def motion_tool():
    print("\n--- LINEAR MOTION ---")
    print("Use constant-acceleration equations: v = u + at, s = ut + 1/2 at²")
    u = number("Initial velocity u (m/s): ")
    a = number("Acceleration a (m/s^2): ")
    t = positive_number("Time t (s): ")
    v = u + a * t
    displacement = u * t + 0.5 * a * t * t
    print(f"Final velocity v = {v:.6g} m/s")
    print(f"Displacement s = {displacement:.6g} m")
    print(f"Average velocity over this interval = {displacement / t:.6g} m/s")
    print(f"Acceleration = {a:.6g} m/s^2")
    print("Distance equals |displacement| only if direction does not reverse.")


def projectile_tool():
    print("\n--- IDEAL PROJECTILE MOTION ---")
    print("Assumptions: launch and landing heights equal; no air resistance.")
    speed = positive_number("Initial speed (m/s): ", allow_zero=False)
    angle = number("Launch angle above horizontal (degrees): ")
    g = positive_number("Gravity magnitude (m/s^2, normally 9.81): ", allow_zero=False)
    theta = math.radians(angle)
    vx = speed * math.cos(theta)
    vy = speed * math.sin(theta)
    time = 2 * vy / g
    height = vy * vy / (2 * g)
    distance = vx * time
    print(f"Horizontal velocity = {vx:.6g} m/s")
    print(f"Vertical velocity = {vy:.6g} m/s")
    print(f"Time of flight = {time:.6g} s")
    print(f"Maximum height above launch point = {height:.6g} m")
    print(f"Horizontal range = {distance:.6g} m")


def force_tool():
    print("\n--- NEWTON'S SECOND LAW ---")
    mass = positive_number("Mass (kg): ", allow_zero=False)
    acceleration = vector_input("acceleration (m/s^2)")
    force = vector_scale(acceleration, mass)
    print("Net force F = m a =", fmt_vector(force), "N")
    print(f"Net-force magnitude = {magnitude(force):.6g} N")
    print("This gives net force; individual forces require a free-body diagram.")


def energy_tool():
    print("\n--- WORK, KINETIC ENERGY & POWER ---")
    mass = positive_number("Mass (kg): ", allow_zero=False)
    speed = positive_number("Speed (m/s): ")
    force = positive_number("Constant force magnitude (N): ")
    distance = positive_number("Displacement magnitude (m): ")
    angle = number("Angle between force and displacement (degrees): ")
    kinetic = 0.5 * mass * speed * speed
    work = force * distance * math.cos(math.radians(angle))
    print(f"Kinetic energy = {kinetic:.6g} J")
    print(f"Work by the force = {work:.6g} J")
    time = positive_number("Time taken for this work (s; enter 0 if unknown): ")
    if time > 0:
        print(f"Average power for this work = {work / time:.6g} W")
    else:
        print("Power not calculated because time was not provided.")


def circular_tool():
    print("\n--- CIRCULAR MOTION ---")
    radius = positive_number("Radius (m): ", allow_zero=False)
    speed = positive_number("Speed (m/s): ")
    print(f"Angular speed omega = v/r = {speed / radius:.6g} rad/s")
    print(f"Centripetal acceleration = v²/r = {speed**2 / radius:.6g} m/s^2")
    mass = positive_number("Mass (kg; enter 0 to skip centripetal force): ")
    if mass > 0:
        print(f"Required net inward force = mv²/r = {mass * speed**2 / radius:.6g} N")


def matrix_tool():
    print("\n--- 2x2 MATRIX TOOL ---")
    print("Enter matrix A row by row:")
    a = [[number("A[1,1] = "), number("A[1,2] = ")],
         [number("A[2,1] = "), number("A[2,2] = ")]]
    print("Enter matrix B row by row:")
    b = [[number("B[1,1] = "), number("B[1,2] = ")],
         [number("B[2,1] = "), number("B[2,2] = ")]]
    print("A + B =")
    for row in range(2):
        print([a[row][col] + b[row][col] for col in range(2)])
    print("A × B =")
    for i in range(2):
        print([sum(a[i][k] * b[k][j] for k in range(2)) for j in range(2)])
    det = a[0][0] * a[1][1] - a[0][1] * a[1][0]
    print(f"det(A) = {det:.6g}")
    if abs(det) > EPSILON:
        inv = [[a[1][1] / det, -a[0][1] / det],
               [-a[1][0] / det, a[0][0] / det]]
        print("Inverse of A =")
        for row in inv:
            print([round(x, 6) for x in row])
    else:
        print("A has no inverse because its determinant is zero.")


def quadratic_tool():
    print("\n--- QUADRATIC EQUATION ---")
    print("Solve ax² + bx + c = 0")
    a = number("a = ")
    b = number("b = ")
    c = number("c = ")
    if abs(a) < EPSILON:
        if abs(b) < EPSILON:
            print("Not a unique linear equation." if abs(c) < EPSILON else "No solution.")
        else:
            print(f"Linear solution x = {-c / b:.6g}")
        return
    discriminant = b * b - 4 * a * c
    print(f"Discriminant D = {discriminant:.6g}")
    if discriminant > EPSILON:
        root = math.sqrt(discriminant)
        print(f"x1 = {(-b + root) / (2*a):.6g}")
        print(f"x2 = {(-b - root) / (2*a):.6g}")
    elif abs(discriminant) <= EPSILON:
        print(f"Repeated root x = {-b / (2*a):.6g}")
    else:
        real = -b / (2 * a)
        imag = math.sqrt(-discriminant) / abs(2 * a)
        print(f"x1 = {real:.6g} + {imag:.6g}i")
        print(f"x2 = {real:.6g} - {imag:.6g}i")


def statistics_tool():
    print("\n--- STATISTICS ---")
    while True:
        raw = input("Enter data values separated by spaces or commas: ").replace(",", " ").split()
        try:
            values = [float(x) for x in raw]
            if not values or not all(math.isfinite(x) for x in values):
                raise ValueError
            break
        except ValueError:
            print("Enter at least one valid finite number.")
    print(f"Count = {len(values)}")
    print(f"Mean = {statistics.mean(values):.6g}")
    print(f"Median = {statistics.median(values):.6g}")
    print(f"Minimum = {min(values):.6g}")
    print(f"Maximum = {max(values):.6g}")
    print(f"Range = {max(values) - min(values):.6g}")
    if len(values) >= 2:
        print(f"Population standard deviation = {statistics.pstdev(values):.6g}")
        print(f"Sample standard deviation = {statistics.stdev(values):.6g}")
    else:
        print("Standard deviation needs at least two values for the sample measure.")


def unit_conversion_tool():
    print("\n--- UNIT CONVERSIONS ---")
    print("1 km/h to m/s   2 m/s to km/h   3 degrees to radians")
    print("4 radians to degrees   5 Celsius to Fahrenheit   6 Fahrenheit to Celsius")
    choice = input("Choice: ").strip()
    value = number("Value: ")
    conversions = {
        "1": (value / 3.6, "m/s"),
        "2": (value * 3.6, "km/h"),
        "3": (math.radians(value), "rad"),
        "4": (math.degrees(value), "degrees"),
        "5": (value * 9 / 5 + 32, "°F"),
        "6": ((value - 32) * 5 / 9, "°C"),
    }
    if choice in conversions:
        result, unit = conversions[choice]
        print(f"Result = {result:.8g} {unit}")
    else:
        print("Unknown conversion choice.")


def main():
    tools = {
        "1": ("Vectors and direction", vector_tool),
        "2": ("Target direction and robot orientation", target_direction_tool),
        "3": ("Linear motion", motion_tool),
        "4": ("Projectile motion", projectile_tool),
        "5": ("Force from mass and acceleration", force_tool),
        "6": ("Work, energy and power", energy_tool),
        "7": ("Circular motion", circular_tool),
        "8": ("2x2 matrices", matrix_tool),
        "9": ("Quadratic equations", quadratic_tool),
        "10": ("Statistics and range", statistics_tool),
        "11": ("Unit conversions", unit_conversion_tool),
    }
    print("=" * 58)
    print("VICTOR — Physics, Mathematics & Robotics Assistant")
    print("Vector • Intelligence • Calculations • Trajectory • Orientation")
    print("=" * 58)
    while True:
        print("\nMAIN MENU")
        for key, (label, _) in tools.items():
            print(f"{key:>2}. {label}")
        print(" 0. Exit")
        choice = input("Choose a tool: ").strip()
        if choice == "0":
            print("Victor: Session ended. Keep exploring and learning!")
            break
        if choice in tools:
            try:
                tools[choice][1]()
            except (ValueError, ZeroDivisionError) as exc:
                print("Calculation could not be completed:", exc)
        else:
            print("Unknown choice. Select one of the listed numbers.")


if __name__ == "__main__":
    main()
