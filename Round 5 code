from datamodel import Order, OrderDepth, TradingState
from typing import Dict, List, Optional


# Round 5 relative-value strategy.
# The products are built in families of five.  Within a family, the common move is
# mostly noise for us; the tradeable edge is when one member is too high/low versus
# the family average plus its normal offset.
#
# Important: I am intentionally NOT using a live EMA for the offsets.  It looked
# sensible, but it moved the fair value toward the bad price and destroyed the
# signal.  These offsets are the latest-regime calibration from the provided data.

POSITION_LIMIT = 10  # Change only if the Round 5 UI explicitly shows a higher limit.

PRODUCT_GROUPS = {
    "GALAXY_SOUNDS": [
        "GALAXY_SOUNDS_BLACK_HOLES",
        "GALAXY_SOUNDS_DARK_MATTER",
        "GALAXY_SOUNDS_PLANETARY_RINGS",
        "GALAXY_SOUNDS_SOLAR_FLAMES",
        "GALAXY_SOUNDS_SOLAR_WINDS",
    ],
    "MICROCHIP": [
        "MICROCHIP_CIRCLE",
        "MICROCHIP_OVAL",
        "MICROCHIP_RECTANGLE",
        "MICROCHIP_SQUARE",
        "MICROCHIP_TRIANGLE",
    ],
    "OXYGEN": [
        "OXYGEN_SHAKE_CHOCOLATE",
        "OXYGEN_SHAKE_EVENING_BREATH",
        "OXYGEN_SHAKE_GARLIC",
        "OXYGEN_SHAKE_MINT",
        "OXYGEN_SHAKE_MORNING_BREATH",
    ],
    "PANEL": ["PANEL_1X2", "PANEL_1X4", "PANEL_2X2", "PANEL_2X4", "PANEL_4X4"],
    "PEBBLES": ["PEBBLES_L", "PEBBLES_M", "PEBBLES_S", "PEBBLES_XL", "PEBBLES_XS"],
    "ROBOT": ["ROBOT_DISHES", "ROBOT_IRONING", "ROBOT_LAUNDRY", "ROBOT_MOPPING", "ROBOT_VACUUMING"],
    "SLEEP": ["SLEEP_POD_COTTON", "SLEEP_POD_LAMB_WOOL", "SLEEP_POD_NYLON", "SLEEP_POD_POLYESTER", "SLEEP_POD_SUEDE"],
    "SNACKPACK": ["SNACKPACK_CHOCOLATE", "SNACKPACK_PISTACHIO", "SNACKPACK_RASPBERRY", "SNACKPACK_STRAWBERRY", "SNACKPACK_VANILLA"],
    "TRANSLATOR": [
        "TRANSLATOR_ASTRO_BLACK",
        "TRANSLATOR_ECLIPSE_CHARCOAL",
        "TRANSLATOR_GRAPHITE_MIST",
        "TRANSLATOR_SPACE_GRAY",
        "TRANSLATOR_VOID_BLUE",
    ],
    "UV_VISOR": ["UV_VISOR_AMBER", "UV_VISOR_MAGENTA", "UV_VISOR_ORANGE", "UV_VISOR_RED", "UV_VISOR_YELLOW"],
}

# Normal distance of each product from its own family average.
FAIR_OFFSET = {
    "GALAXY_SOUNDS_BLACK_HOLES": 1554.620220,
    "GALAXY_SOUNDS_DARK_MATTER": -911.505430,
    "GALAXY_SOUNDS_PLANETARY_RINGS": -386.132230,
    "GALAXY_SOUNDS_SOLAR_FLAMES": -135.553230,
    "GALAXY_SOUNDS_SOLAR_WINDS": -121.429330,
    "MICROCHIP_CIRCLE": 29.080730,
    "MICROCHIP_OVAL": -3298.847720,
    "MICROCHIP_RECTANGLE": -1220.396570,
    "MICROCHIP_SQUARE": 5404.055230,
    "MICROCHIP_TRIANGLE": -913.891670,
    "OXYGEN_SHAKE_CHOCOLATE": -182.974560,
    "OXYGEN_SHAKE_EVENING_BREATH": -827.389210,
    "OXYGEN_SHAKE_GARLIC": 2656.610390,
    "OXYGEN_SHAKE_MINT": -898.005560,
    "OXYGEN_SHAKE_MORNING_BREATH": -748.241060,
    "PANEL_1X2": -438.929510,
    "PANEL_1X4": -1156.800310,
    "PANEL_2X2": -938.746760,
    "PANEL_2X4": 2181.065490,
    "PANEL_4X4": 353.411090,
    "PEBBLES_L": 458.079850,
    "PEBBLES_M": 871.802400,
    "PEBBLES_S": -2050.914250,
    "PEBBLES_XL": 4664.916700,
    "PEBBLES_XS": -3943.884700,
    "ROBOT_DISHES": 1057.282530,
    "ROBOT_IRONING": -1722.997620,
    "ROBOT_LAUNDRY": -550.790670,
    "ROBOT_MOPPING": 2169.209330,
    "ROBOT_VACUUMING": -952.703570,
    "SLEEP_POD_COTTON": 822.006770,
    "SLEEP_POD_LAMB_WOOL": -816.534730,
    "SLEEP_POD_NYLON": -1501.840030,
    "SLEEP_POD_POLYESTER": 1133.873620,
    "SLEEP_POD_SUEDE": 362.494370,
    "SNACKPACK_CHOCOLATE": -390.561110,
    "SNACKPACK_PISTACHIO": -714.589510,
    "SNACKPACK_RASPBERRY": -8.002410,
    "SNACKPACK_STRAWBERRY": 970.738140,
    "SNACKPACK_VANILLA": 142.414890,
    "TRANSLATOR_ASTRO_BLACK": -745.921980,
    "TRANSLATOR_ECLIPSE_CHARCOAL": -144.481480,
    "TRANSLATOR_GRAPHITE_MIST": 550.863920,
    "TRANSLATOR_SPACE_GRAY": -961.176930,
    "TRANSLATOR_VOID_BLUE": 1300.716470,
    "UV_VISOR_AMBER": -3465.562730,
    "UV_VISOR_MAGENTA": 1227.522920,
    "UV_VISOR_ORANGE": 477.843720,
    "UV_VISOR_RED": 1437.592620,
    "UV_VISOR_YELLOW": 322.603470,
}

# Products not listed here are deliberately skipped.  They either had weak edge or
# gave negative contribution in the latest validation slice.
SIGNAL_THRESHOLD = {
    "ROBOT_DISHES": 10,
    "SNACKPACK_CHOCOLATE": 300,
    "SLEEP_POD_POLYESTER": 300,
    "PANEL_2X4": 750,
    "PEBBLES_L": 10,
    "OXYGEN_SHAKE_CHOCOLATE": 500,
    "ROBOT_VACUUMING": 300,
    "SNACKPACK_VANILLA": 25,
    "SNACKPACK_PISTACHIO": 10,
    "SNACKPACK_STRAWBERRY": 300,
    "OXYGEN_SHAKE_EVENING_BREATH": 300,
    "GALAXY_SOUNDS_SOLAR_WINDS": 400,
    "SNACKPACK_RASPBERRY": 25,
    "GALAXY_SOUNDS_BLACK_HOLES": 750,
    "TRANSLATOR_VOID_BLUE": 10,
    "MICROCHIP_TRIANGLE": 750,
    "GALAXY_SOUNDS_SOLAR_FLAMES": 25,
    "SLEEP_POD_SUEDE": 400,
    "SLEEP_POD_NYLON": 400,
    "UV_VISOR_YELLOW": 1500,
    "UV_VISOR_MAGENTA": 750,
    "OXYGEN_SHAKE_MINT": 750,
    "OXYGEN_SHAKE_MORNING_BREATH": 500,
    "PANEL_2X2": 10,
    "PEBBLES_XS": 1000,
    "TRANSLATOR_ECLIPSE_CHARCOAL": 500,
    "UV_VISOR_ORANGE": 10,
    "SLEEP_POD_COTTON": 300,
    "MICROCHIP_SQUARE": 10,
    "MICROCHIP_OVAL": 1000,
    "ROBOT_IRONING": 200,
    "GALAXY_SOUNDS_DARK_MATTER": 750,
    "PEBBLES_M": 100,
    "PANEL_1X4": 300,
    "PEBBLES_S": 10,
    "UV_VISOR_RED": 10,
    "OXYGEN_SHAKE_GARLIC": 1000,
    "ROBOT_MOPPING": 100,
    "ROBOT_LAUNDRY": 150,
    "UV_VISOR_AMBER": 150,
    "TRANSLATOR_ASTRO_BLACK": 10,
    "TRANSLATOR_SPACE_GRAY": 1000,
    "GALAXY_SOUNDS_PLANETARY_RINGS": 750,
    "PEBBLES_XL": 10,
}


class Trader:
    def _mid_price(self, depth: OrderDepth) -> Optional[float]:
        if not depth.buy_orders or not depth.sell_orders:
            return None
        return (max(depth.buy_orders.keys()) + min(depth.sell_orders.keys())) / 2.0

    def _orders_to_target(self, product: str, depth: OrderDepth, current_pos: int, target_pos: int) -> List[Order]:
        target_pos = max(-POSITION_LIMIT, min(POSITION_LIMIT, target_pos))
        delta = target_pos - current_pos
        orders: List[Order] = []

        if delta > 0:
            # Need to buy.  Cross the visible asks from cheapest upward.
            remaining = delta
            for price, volume in sorted(depth.sell_orders.items()):
                take = min(remaining, abs(volume))
                if take > 0:
                    orders.append(Order(product, price, take))
                    remaining -= take
                if remaining == 0:
                    break

        elif delta < 0:
            # Need to sell.  Cross the visible bids from highest downward.
            remaining = -delta
            for price, volume in sorted(depth.buy_orders.items(), reverse=True):
                take = min(remaining, abs(volume))
                if take > 0:
                    orders.append(Order(product, price, -take))
                    remaining -= take
                if remaining == 0:
                    break

        return orders

    def run(self, state: TradingState):
        result: Dict[str, List[Order]] = {}

        for group_name, products in PRODUCT_GROUPS.items():
            mids: Dict[str, float] = {}

            # Do not average a partial family.  If one book disappears, the family
            # average is biased and every signal in that family becomes polluted.
            for product in products:
                depth = state.order_depths.get(product)
                if depth is None:
                    mids = {}
                    break
                mid = self._mid_price(depth)
                if mid is None:
                    mids = {}
                    break
                mids[product] = mid

            if len(mids) != len(products):
                continue

            group_mid = sum(mids.values()) / len(products)

            for product in products:
                threshold = SIGNAL_THRESHOLD.get(product)
                if threshold is None:
                    continue

                current_pos = state.position.get(product, 0)
                fair_mid = group_mid + FAIR_OFFSET[product]
                residual = mids[product] - fair_mid

                # Expensive versus family -> short.  Cheap versus family -> long.
                # If the signal is in the middle, we keep the existing inventory;
                # forcing a flat exit here was the bug in the bad fixed version.
                target_pos = current_pos
                if residual > threshold:
                    target_pos = -POSITION_LIMIT
                elif residual < -threshold:
                    target_pos = POSITION_LIMIT

                depth = state.order_depths[product]
                product_orders = self._orders_to_target(product, depth, current_pos, target_pos)
                if product_orders:
                    result[product] = product_orders

        conversions = 0
        trader_data = "r5_static_relative_value_v3"
        return result, conversions, trader_data
