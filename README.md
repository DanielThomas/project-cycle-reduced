Demonstration of external/project dependency cycles for https://github.com/gradle/gradle/issues/4785.

To reproduce, run either:

- `./gradlew :c:dependencies`
- `./gradlew :c:printCompile`

And observe that the external dependency is selected, and is used in preference to treating it as a project dependency and failing with a cyclic dependency:

	compile - Dependencies for source set 'main' (deprecated, use 'implementation ' instead).
	+--- project :a -> netflix:a:1.0
	\--- project :b
	     \--- netflix:other:1.0
	          \--- netflix:a:1.0

The module selection output will read `Selected netflix:a:1.0 from conflicting modules [netflix:a:unspecified, netflix:a:1.0]`:

	16:19:43.519 [DEBUG] [org.gradle.api.internal.artifacts.repositories.resolver.ExternalResourceResolver] Metadata file found for module 'netflix:a:1.0' in repository 'maven'.
	16:19:43.519 [DEBUG] [org.gradle.api.internal.artifacts.ivyservice.modulecache.DefaultModuleMetadataCache] Recording module descriptor in cache: netflix:a:1.0 [changing = false]
	16:19:43.519 [DEBUG] [org.gradle.api.internal.artifacts.ivyservice.ivyresolve.RepositoryChainComponentMetaDataResolver] Using netflix:a:1.0 from Maven repository 'maven'
	16:19:43.519 [DEBUG] [org.gradle.api.internal.artifacts.ivyservice.resolveengine.graph.conflicts.DefaultConflictHandler] Selected netflix:a:1.0 from conflicting modules [netflix:a:unspecified, netflix:a:1.0].
	
Configuring a version that is higher on the project results in:

	compile - Dependencies for source set 'main' (deprecated, use 'implementation ' instead).
	+--- project :a
	|    \--- project :b
	|         \--- netflix:other:1.0
	|              \--- netflix:a:1.0 -> project :a (*)
	\--- project :b (*)

In rare case we've seen these cycles cause outright dependency resolution failures:

	Caused by: java.lang.RuntimeException: Problems reading data from Binary store in /mnt/builds/slave/workspace/CPIE-platform-legacy-release/.gradle/.nebulatmp/gradle3198753972717668990.bin (exist: true)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.store.DefaultBinaryStore$SimpleBinaryData.read(DefaultBinaryStore.java:129)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder$6.create(TransientConfigurationResultsBuilder.java:133)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder$6.create(TransientConfigurationResultsBuilder.java:130)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.store.CachedStoreFactory$SimpleStore.load(CachedStoreFactory.java:97)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder.load(TransientConfigurationResultsBuilder.java:130)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsLoader.create(TransientConfigurationResultsLoader.java:34)
		at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.loadTransientGraphResults(DefaultLenientConfiguration.java:160)
		at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelNodes(DefaultLenientConfiguration.java:173)
		at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelModuleDependencies(DefaultLenientConfiguration.java:165)
		at org.gradle.api.internal.artifacts.ivyservice.DefaultLenientConfiguration.getFirstLevelModuleDependencies(DefaultLenientConfiguration.java:282)
		at org.gradle.api.internal.artifacts.ivyservice.DefaultResolvedConfiguration.getFirstLevelModuleDependencies(DefaultResolvedConfiguration.java:67)
		at org.gradle.api.internal.artifacts.ivyservice.ErrorHandlingConfigurationResolver$ErrorHandlingResolvedConfiguration.getFirstLevelModuleDependencies(ErrorHandlingConfigurationResolver.java:280)
		... 67 more
	Caused by: java.lang.IllegalStateException: Unexpected parent dependency id 1562. Seen ids: [2049, 2, 2050, 2052, 2054, 2057, 2058, 2059, 2064, 2073, 2074, 2075, 2076, 2077, 2081, 2082, 2093, 2097, 2105, 2114, 2116, 2117, 2122, 2124, 2125, 2127, 2133, 2143, 2144, 2152, 2165, 2166, 2172, 2173, 2183, 2186, 2189, 2192, 2211, 2212, 2213, 2214, 2215, 2216, 2217, 2218, 2219, 2240, 2241, 2242, 2245, 2252, 2253, 2258, 2259, 2267, 2268, 2269, 2303, 2304, 2305, 2306, 2307, 2308, 2309, 2310, 2311, 2312, 2313, 2314, 2315, 2316, 2323, 2326, 2333, 2334, 2335, 2340, 2341, 2345, 2346, 2347, 2350, 2351, 2360, 2361, 2362, 2363, 2364, 2365, 2379, 2380, 2381, 2382, 2383, 2384, 2387, 2394, 2395, 2406, 2407, 2408, 2409, 2416, 2420, 2421, 2422, 2455, 2456, 2457, 2458, 2459, 2471, 2472, 2486, 2487, 2502, 2503, 2510, 2511, 2521, 2522, 2525, 2526, 2527, 2528, 2529, 2530, 2537, 2538, 2540, 2541, 2542, 2543, 2544, 2545, 2546, 2547, 2554, 2555, 2556, 2557, 2558, 2559, 2560, 2561, 1012, 1013, 1014, 1015, 1016, 1017, 1018, 1019, 1020, 1021, 1022, 1023, 1024, 1025, 1026, 1027, 1028, 1029, 1030, 1031, 1032, 1033, 1034, 1035, 1036, 1037, 1038, 1039, 1040, 1041, 1042, 1043, 1044, 1045, 1046, 1047, 1048, 1049, 1050, 1051, 1052, 1053, 1054, 1055, 1056, 1057, 1058, 1059, 1060, 1061, 1062, 1063, 1064, 1065, 1066, 1067, 1068, 1069, 1070, 1071, 1072, 1073, 1074, 1075, 1076, 1077, 1078, 1079, 1080, 1081, 1082, 1083, 1084, 1085, 1086, 1087, 1088, 1089, 1090, 1091, 1092, 1093, 1094, 1095, 1096, 1097, 1098, 1099, 1100, 1101, 1102, 1103, 1104, 1105, 1106, 1107, 1108, 1109, 1110, 1111, 1112, 1113, 1114, 1115, 1116, 1117, 1118, 1119, 1120, 1121, 1122, 1123, 1124, 1125, 1126, 1127, 1128, 1129, 1130, 1131, 1132, 1133, 1134, 1135, 1136, 1137, 1138, 1139, 1140, 1141, 1142, 1143, 1144, 1145, 1146, 1147, 1148, 1149, 1150, 1151, 1152, 1153, 1154, 1155, 1156, 1157, 1158, 1159, 1160, 1161, 1162, 1163, 1164, 1165, 1166, 1167, 1168, 1169, 1170, 1171, 1172, 1173, 1174, 1175, 1176, 1177, 1178, 1179, 1180, 1181, 1182, 1183, 1184, 1185, 1186, 1187, 1188, 1189, 1190, 1191, 1192, 1193, 1194, 1195, 1196, 1197, 1198, 1199, 1200, 1201, 1202, 1203, 1204, 1205, 1206, 1207, 1208, 1209, 1210, 1211, 1212, 1213, 1214, 1215, 1216, 1217, 1218, 1219, 1220, 1221, 1222, 1223, 1224, 1225, 1226, 1227, 1228, 1229, 1230, 1231, 1232, 1233, 1234, 1235, 1236, 1237, 1238, 1239, 1240, 1241, 1242, 1243, 1244, 1245, 1246, 1247, 1248, 1249, 1250, 1251, 1252, 1253, 1254, 1255, 1256, 1257, 1258, 1259, 1261, 1262, 1263, 1264, 1265, 1266, 1267, 1268, 1269, 1270, 1271, 1272, 1273, 1274, 1275, 1276, 1277, 1278, 1279, 1280, 1281, 1282, 1283, 1284, 1285, 1286, 1287, 1288, 1290, 1291, 1292, 1293, 1294, 1295, 1296, 1297, 1298, 1299, 1300, 1301, 1302, 1303, 1304, 1305, 1306, 1307, 1308, 1309, 1312, 1313, 1314, 1315, 1316, 1317, 1318, 1319, 1320, 1321, 1322, 1323, 1324, 1326, 1327, 1328, 1329, 1330, 1331, 1332, 1333, 1334, 1335, 1337, 1338, 1339, 1340, 1341, 1343, 1344, 1345, 1346, 1347, 1348, 1349, 1350, 1351, 1352, 1353, 1354, 1355, 1356, 1357, 1358, 1359, 1361, 1362, 1363, 1364, 1365, 1366, 1367, 1368, 1369, 1370, 1371, 1372, 1373, 1375, 1376, 1377, 1378, 1379, 1380, 1381, 1382, 1383, 1384, 1385, 1386, 1387, 1388, 1389, 1390, 1391, 1392, 1393, 1394, 1395, 1396, 1397, 1398, 1399, 1400, 1401, 1402, 1403, 1404, 1405, 1406, 1407, 1409, 1410, 1411, 1412, 1413, 1414, 1415, 1416, 1417, 1418, 1419, 1420, 1421, 1422, 1423, 1424, 1425, 1427, 1428, 1429, 1430, 1431, 1432, 1433, 1434, 1435, 1436, 1437, 1438, 1439, 1440, 1441, 1442, 1443, 1444, 1446, 1447, 1448, 1449, 1452, 1453, 1454, 1455, 1456, 1457, 1458, 1459, 1460, 1461, 1462, 1463, 1464, 1465, 1466, 1467, 1468, 1469, 1470, 1471, 1472, 1473, 1474, 1476, 1477, 1478, 1479, 1481, 1482, 1483, 1484, 1485, 1486, 1487, 1488, 1490, 1491, 1492, 1493, 1494, 1495, 1496, 1531, 1532, 1533, 1534, 1544, 1545, 1546, 1547, 1553, 1554, 1555, 1558, 1559, 1578, 1579, 1580, 1581, 1588, 1589, 1590, 1598, 1599, 1600, 1603, 1624, 1625, 1626, 1633, 1634, 1639, 1640, 1645, 1646, 1652, 1653, 1654, 1655, 1656, 1657, 1658, 1659, 1661, 1662, 1663, 1665, 1666, 1667, 1668, 1669, 1670, 1672, 1673, 1680, 1681, 1682, 1686, 1690, 1691, 1692, 1693, 1694, 1695, 1697, 1698, 1699, 1700, 1707, 1708, 1709, 1710, 1711, 1712, 1713, 1721, 1722, 1723, 1724, 1725, 1726, 1727, 1728, 1730, 1731, 1732, 1733, 1734, 1735, 1736, 1737, 1738, 1739, 1740, 1741, 1742, 1743, 1744, 1745, 1746, 1747, 1748, 1749, 1750, 1751, 1752, 1753, 1754, 1755, 1756, 1757, 1758, 1759, 1760, 1761, 1762, 1763, 1764, 1765, 1766, 1767, 1768, 1769, 1770, 1771, 1772, 1773, 1774, 1775, 1776, 1777, 1778, 1779, 1780, 1781, 1782, 1783, 1784, 1785, 1786, 1787, 1788, 1789, 1790, 1791, 1792, 1793, 1794, 1795, 1796, 1797, 1798, 1799, 1800, 1801, 1802, 1803, 1804, 1805, 1806, 1807, 1808, 1809, 1810, 1811, 1812, 1813, 1814, 1815, 1816, 1817, 1818, 1819, 1820, 1821, 1822, 1823, 1824, 1825, 1826, 1827, 1828, 1829, 1830, 1831, 1832, 1833, 1834, 1835, 1836, 1837, 1838, 1839, 1840, 1841, 1842, 1843, 1844, 1845, 1846, 1847, 1848, 1849, 1850, 1851, 1852, 1857, 1858, 1861, 1862, 1863, 1873, 1874, 1875, 1876, 1877, 1878, 1881, 1882, 1884, 1885, 1886, 1887, 1888, 1890, 1891, 1892, 1893, 1894, 1895, 1896, 1897, 1907, 1910, 1911, 1912, 1915, 1916, 1917, 1937, 1938, 1941, 1942, 1943, 1944, 1946, 1947, 1948, 1951, 1957, 1958, 1960, 1961, 1962, 1964, 1965, 1966, 1967, 1968, 1974, 1975, 1976, 1981, 1982, 1983, 1986, 1992, 1994, 1995, 2005, 2006, 2007, 2011, 2012, 2013, 2014, 2016, 2017, 2020, 2021, 2022, 2027, 2031, 2032, 2033, 2036, 2037, 2038, 2039]
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder.deserialize(TransientConfigurationResultsBuilder.java:192)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder.access$200(TransientConfigurationResultsBuilder.java:49)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder$6$1.read(TransientConfigurationResultsBuilder.java:135)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.oldresult.TransientConfigurationResultsBuilder$6$1.read(TransientConfigurationResultsBuilder.java:133)
		at org.gradle.api.internal.artifacts.ivyservice.resolveengine.store.DefaultBinaryStore$SimpleBinaryData.read(DefaultBinaryStore.java:127)
		... 78 more
