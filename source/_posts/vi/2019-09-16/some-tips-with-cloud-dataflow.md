---
title: Một vài note khi sử dụng Cloud Dataflow
tags:
  - it
  - gcp
  - dataflow
category: it
lang: vi
date: 2019-09-16 04:05:35
---
### 1. --region=REGION để chỉ định Region cho Job Dataflow
Khi deploy 1 job lên dataflow, nếu không chỉ định region, nó sẽ được **mặc định** deploy lên region us-central1, tức là Instance chứa Job Dataflow của bạn sẽ đặt ở 1 Data center nằm ở Mỹ. Điều này có thể không ảnh hưởng lắm, cho đến khi bạn cần xử lý lượng dữ liệu nhiều hơn, và trong xử lý có phát sinh các I/O với các service khác. 

Ví dụ, sau khi tính toán và xử lý dữ liệu xong, thì lưu kết quả vào MySQL.Trong khi xử lý dữ liệu thì cũng phải tham chiếu 1 số thông tin từ master trên MySQL. Instance của Job dataflow nằm ở us-central1 (Mỹ), còn instance của MySQL nằm ở asia-northeast1 (Tokyo). Điều này sẽ ảnh hưởng khá lớn đến hiệu năng của xử lý của bạn, vì việc truyền tải dữ liệu giữa Nhật và Mỹ chắc chắn sẽ không thể nhanh bằng Nhật - Nhật rồi . Đừng quên đặt các instance ở cùng region cho các xử lý có phát sinh I/O nhé. 


### 2. Sử dụng câu lệnh Gcloud dataflow để cancel job.

Trước hết, gõ cái này để lấy ra list job trong Project.

 ```
 gcloud dataflow jobs list
 ```

Sau đó, bạn có thể dùng câu lệnh này để cancel 1 job Dataflow.

```
gcloud dataflow jobs cancel JOB_ID --region REGION` 
```

Lại 1 lần nữa có liên quan đến Region. Nếu không chỉ định parameter --region cho câu lệnh bên trên, thì Dataflow nó sẽ tự chỉ định region us-central1. Nếu job của chúng ta đang nằm ở asia-northeast1, thì câu lệnh trên sẽ không được thực hiện và sẽ có lỗi sau: 

> Failed to cancel job [XXXYYYZZZ]: (4a2b1f4c989a438d): Could not cancel workflow; user does not have sufficient permissions on project: AAABBBCCC,or the job does not exist in the project. Please ensure you have permission to access the job and the `--region` flag, us-central1, matches the job's region.

### 3. Hiện tại, không có cách nào để **Delete** 1 job, mà chỉ có thể **Cancel** nó đi thôi.

Mình cũng search google xem có cách nào xoá hẳn 1 job khỏi List Jobs trên UI của Cloud Datafow không. Nhưng rất tiếc là không có cách nào cả. 
Bạn chỉ có thể cancel 1 job nào đó, và job đó sẽ vẫn tồn tại trong list, với Status là **Canceled**. Nhìn cái list có nhiều Job deloy xong bị lỗi, phải cancel đi, mà không xoá được hẳn nó khỏi list, thật là ức chế ...

Nhận tiện thì cũng có khá nhiều người ức chế với việc này =))
[Here](https://googlecloudplatform.uservoice.com/forums/302628-cloud-dataflow/suggestions/10099404-a-way-to-delete-old-runs-of-dataflow-jobs-search). Cái issue này tồn tại 4 năm rồi, và phía Dataflow có vẻ không có ý định cho thêm tính năng này. 

### 4. Về bản chất thì Job Dataflow cũng được deloy và chạy trên những VM instance.
Thế nên bạn hoàn toàn có thể vào mục **Compute Engine**, và sẽ thấy 1, hoặc nhiều con máy VM ở đó. Ở cột In use by sẽ có tên của Dataflow Job Name ABC, ý là con máy VM này đang được dùng bởi Dataflow Job ABC.

Tuy nó là 1 VM, nhưng bạn không thể hẹn giờ bật tắt nó như những VM thông thường khác (để tiết kiệm chi phí những lúc không dùng đến chẳng hạn), nếu Dataflow Job type của bạn là Streaming. 
Để đảm bảo cho việc nhận và xử lý dữ liệu mọi lúc mọi nơi, không thể stop instance chứa Job Dataflow được. 
Instance trên GCE sẽ bị xoá hoàn toàn khi bạn thực hiện việc Cancel job. 

Ngoài ra, giống như thuộc tính của VM khác, khi deploy Job Dataflow, bạn hoàn toàn có thể chỉ định Network, Subnetwork cho nó. 
Cho ai cần tham khảo chi tiết: [here](https://cloud.google.com/dataflow/docs/guides/specifying-networks)

Nếu không chỉ định gì, giống như những VM khác, nó sẽ tự động chui vào Network **Default**.
Có thể dùng 1 trong 2 cách dưới đây để chỉ định Subnetwork:
- Complete URL: 
```
https://www.googleapis.com/compute/v1/projects/<HOST_PROJECT>/regions/<REGION>/subnetworks/<SUBNETWORK>
```
- Short form:
```
regions/<REGION>/subnetworks/<SUBNETWORK>
```

và thêm parameter: `--subnetwork=SUB_NETWORK`
khi deploy job.

Tạm thời thế đã, bài viết hôm nay xin phép dừng tại đây. Hi vọng chia sẻ này sẽ có chút hữu ích với bạn (nếu bạn là newbie và đang vọc vạch với Dataflow)

Còn nếu bạn muốn biết chi tiết hơn, thì mọi thứ đều có ở [here](https://cloud.google.com/dataflow/).

See you.