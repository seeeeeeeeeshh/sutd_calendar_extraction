# SUTD Calendar Extraction

This script extracts your class schedule from SUTD’s MyPortal and generates an `.ics` calendar file that can be imported into Google Calendar, Outlook, or any other calendar app.  

> **Note**: This is an edited version of the original code found at [Pastebin](https://pastebin.com/wSiP2Ljm).

---

 HOW TO USE:
- Go to MyPortal > My Record > My Weekly Schedule.
- Under display options, make sure all 7 of the days and "Show Class Title" are checked. Also make sure that "Show AM/PM" and "Show Instructors" are unchecked. Click the "Refresh Calendar" button if you make any changes.
- Also make sure that the current week is being displayed.
- Open the developer tools and open the console. If you don't know how, you can check the link below. The shortcut key is Ctrl + Shift + I for Windows Users.
- [https://webmasters.stackexchange.com/questions/8525/how-do-i-open-the-javascript-console-in-different-browsers#answer-77337](https://balsamiq.com/support/faqs/browser-console/)
- Paste the code given below and click enter.
- Some browsers might not allow pasting. Type 'allow pasting', press Enter and then use the code.
- It will take about 30 seconds to extract the classes and create a '.ics' file from it, after which it will prompt you to download a 'schedule.ics' file.
     WARNING: The code will interact with the webpage, so until the file download prompt comes, do not change anything in this tab.
- Download the .ics file and import it to any calendar of your choice.
 
*Use the code below:
```
(async function (doc) {
  let classes = [];
  for (let i = 0; i < 14; i++) {
    const MONTHS = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
 
    const day_from_x = {};
    const days = Array.from(doc.querySelectorAll('#WEEKLY_SCHED_HTMLAREA th'))
      .slice(1)
      .map((e, i) => {
        let [day, month] = e.textContent.split('\n')[1].split(' ');
        const x = Math.round(e.getBoundingClientRect().x);
        day_from_x[x] = i;
        return {
          day: parseInt(day),
          month: MONTHS.indexOf(month) + 1,
          year: 2025,
        };
      });
 
    const removeExtraSpaces = s => s.split(' ').filter(s => s.length > 0).join(' ');
 
    const c = Array.from(doc.querySelectorAll('#WEEKLY_SCHED_HTMLAREA td > span'))
      .map(e => e.parentNode)
      .filter(e => day_from_x[Math.round(e.getBoundingClientRect().x)] !== undefined)
      .map(e => {
        const span = e.childNodes[0].childNodes;
        const day = days[day_from_x[Math.round(e.getBoundingClientRect().x)]];
 
        const formatCode = s => {
          const words = removeExtraSpaces(s).split(' ');
          return words[0] + words.slice(1).join(' ');
        }
 
        return {
          name: removeExtraSpaces(`${formatCode(span[0].textContent)} ${span[2].textContent} ${span[4].textContent}`),
          day,
          time: span[6].textContent.split(' - ').map(s => s.split(':').join('')),
          place: removeExtraSpaces(span[8].textContent),
        }
      });
 
    console.log(`Found ${c.length} classes for Week ${i + 1}`);
    classes.push(...c);
 
    doc.getElementById('DERIVED_CLASS_S_SSR_NEXT_WEEK').click();
    await new Promise(res => setTimeout(res, 2000));
  }
  return classes;
})(document.getElementById('ptifrmtgtframe').contentDocument).then(classes => {
  const now = new Date();
 
  const p = (n, l) => ("000" + n).slice(-l);
  const f = (c, i) => `BEGIN:VEVENT
SUMMARY:${c.name}
DTSTAMP;TZID=Asia/Singapore:${p(now.getUTCFullYear(), 4)}${p(now.getMonth() + 1, 2)}${p(now.getUTCDate(), 2)}T000000
DTSTART;TZID=Asia/Singapore:${p(c.day.year, 4)}${p(c.day.month, 2)}${p(c.day.day, 2)}T${p(c.time[0], 4)}00
DTEND;TZID=Asia/Singapore:${p(c.day.year, 4)}${p(c.day.month, 2)}${p(c.day.day, 2)}T${p(c.time[1], 4)}00
LOCATION:${c.place}
END:VEVENT`;
 
  const icsContent = `BEGIN:VCALENDAR
VERSION:2.0
CALSCALE:GREGORIAN
METHOD:PUBLISH
${classes.map(f).join('\n')}
END:VCALENDAR
`;
 
  let file = new Blob([icsContent], { type: 'text/calendar' });
  const a = document.createElement("a");
  const url = URL.createObjectURL(file);
  a.href = url;
  a.download = 'schedule.ics';
  document.body.appendChild(a);
  a.click();
  setTimeout(function() {
    document.body.removeChild(a);
    window.URL.revokeObjectURL(url);
  }, 0);
});

```
