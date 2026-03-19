# Enforce Strict Input Schemas with Enums and Constraints

Don't rely on description text alone to communicate valid input values. Use JSON Schema constraints to enforce them at the protocol level.

```python
from enum import Enum

class SortOrder(str, Enum):
    ASC = "ascending"
    DESC = "descending"

class DateRange(str, Enum):
    LAST_7_DAYS = "last_7_days"
    LAST_30_DAYS = "last_30_days"
    LAST_90_DAYS = "last_90_days"
    ALL_TIME = "all_time"

@tool
def search_activity(
    space_id: str,
    sort: SortOrder = SortOrder.DESC,
    date_range: DateRange = DateRange.LAST_30_DAYS,
    limit: int = 25  # constrained via Field(ge=1, le=100)
) -> dict:
    """Search member activity in a space."""
```

**What this gives you:**
- Models see the enum values in the schema and pick from them
- Invalid values are caught before execution, producing clear validation errors
- Default values reduce the number of parameters the model must decide on
- Type hints auto-generate JSON Schema via frameworks like FastMCP

**Anti-pattern:** Accepting `str` for everything and hoping the model sends valid values. This leads to silent failures or vague errors deep in business logic.

**Source:** NearForm - "Implementing MCP: Tips, tricks and pitfalls"; modelcontextprotocol.info best practices
