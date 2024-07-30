EntityForging has a setter function called setOneYearInDays, which can change the value of the oneYearInDays state variable. But this is completely unnecessary, since oneYearInDays never changes, and should be a constant.

Reccomendation:
Make oneYearInDays a constance and remove the setter function.