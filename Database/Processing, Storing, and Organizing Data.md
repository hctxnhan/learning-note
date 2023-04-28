# OLTP and OLAP
![[Pasted image 20230428165543.png]]
![[Pasted image 20230428165625.png]]
Thường hệ thống OLAP và OLTP làm việc với nhau, đúng hơn là chúng cần nhau. OLTP thường lưu data clean để tạo ra OLAP data warehouse.
Nếu không có transactional data thì không thể có analyses được.
- **OLTP**: được design để xử lý lượng lớn các transactional data trong real-time. Optimized để xử lý nhiều transaction nhỏ và thường xuyên như: credit card purchases hoặc bank transfers. Mục tiêu chính của OLTP là đảm bảo consistency của data để provide quick response time.
- **OLAP**: được design để xử lý lượng lớn data cho các complex analytical operations. Optimized cho handling các query có aggregating và analyzing tập data lớn như data mining, forecasting, trend analysis.
# Storing data
- **ETL** (Extract > Transform > Load): Extract data từ nhiều nguồn, transform sang một định dạng chung và sau đó load lên target database hoặc storage.
- **ELT** (Extract > Load > Transform): Extract data từ nhiều nguồn, load lên target database hoặc storage và sau đó transform dữ liệ theo nhu cầu.
![[Pasted image 20230428172236.png]]
# Database design
## Data modeling
Là quá trình tạo ra data model cho data được lưu trữ.
1. **Conceptual data model**: mô tả entities, relationships, attributes. (ERD).
![[Pasted image 20230428173405.png]]
2. **Logical data model**: define tables, columns, relationships. (schema).
![[Pasted image 20230428173426.png]]
1. **Physics data model**.
