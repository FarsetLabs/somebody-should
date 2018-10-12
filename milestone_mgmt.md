# Currently recommended milestones

* Town Halls (First Monday of the Month)

# Auto-generation of Town-Hall milestones

```python
from github import Github
from dateutil import relativedelta
import calendar

g = Github(<ACCESS TOKEN FROM YOUR GITHUB DEVELOPER PORTAL>)
ss = g.get_repo("FarsetLabs/somebody-should")


def create_th_milestone(repo, due_date: datetime.datetime):
    milestone = dict(
        title=due_date.strftime("%Y-%m Town Hall"),
        state="open",
        description=due_date.strftime(
            "Issues to be reviewed at the %B %Y Town Hall"),
        due_on=due_date
    )

    # Check for contention
    for m in repo.get_milestones():
        if m.title == milestone['title']:
            raise ValueError("Duplicate Title: Reconsider manually editing")
        if m.due_on == milestone['due_on']:
            raise ValueError(
                "Duplicate Due Date: Reconsider manually editing/deleting")

    repo.create_milestone(**milestone)

def get_townhall_date(year, month):
    c = calendar.Calendar(firstweekday=calendar.SUNDAY)
    monthcal = c.monthdatescalendar(year, month)
    town_hall = [day for week in monthcal for day in week if
                 day.weekday() == calendar.MONDAY and
                 day.month == month][0]
    town_hall = datetime.datetime.combine(town_hall, datetime.time(19))
    return town_hall


this_month = datetime.datetime.now().date()
for i in range(12):
    this_month += relativedelta.relativedelta(months=1)
    this_townhall = get_townhall_date(this_month.year, this_month.month)
    try:
        create_th_milestone(ss, this_townhall)
    except ValueError:
        print(f"{this_townhall}: Already has a milestone")
    print(f"{this_townhall}: Milestone satisfied")
```
