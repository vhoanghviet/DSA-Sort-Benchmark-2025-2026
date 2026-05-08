# DSA-Sort-Benchmark-2025-2026

Họ tên : Đoàn Hoàng Việt
MSSV : 25120469
Lớp : 25CTT6

# Thuật toán cài đặt tốt nhất ở lần chạy đầu tiên và các phương thức tối ưu hóa liên quan, lý giải tại sao phương pháp này tốt nhất trong tất cả các cách cài đặt ở lần 1.

Trong giai đoạn đầu tiên, em đã trọng tâm của việc thiết kế giải thuật được đặt vào hiệu suất tối ưu khi xử lý dữ liệu quy mô lớn lên đến hàng trăm nghìn phần tử. Thay vì sử dụng các phương pháp sắp xếp dựa trên so sánh truyền thống vốn bị giới hạn bởi độ phức tạp lý thuyết O(nlogn), phương pháp Radix Sort đã được lựa chọn làm giải pháp cốt lõi cho cả ba bài toán nhờ khả năng vận hành trong thời gian tuyến tính O(n*k).

Đối với bài toán sắp xếp số nguyên, giải thuật Radix Sort theo hướng LSD đã được triển khai để xử lý dữ liệu qua từng cụm 8 bit. Cách tiếp cận này cho phép hoàn tất quá trình chỉ trong bốn lượt lặp cho một số nguyên 32-bit, đồng thời giải quyết triệt để vấn đề số nguyên có dấu thông qua việc hiệu chỉnh mảng đếm tại byte cuối cùng nhằm đưa các giá trị âm về đúng vị trí trong dãy không giảm.

###
void radix_sort(int* a, int n) {
    unsigned int* ua = (unsigned int*)a;
    unsigned int* ut = (unsigned int*)tmp;

    for (int shift = 0; shift < 32; shift += 8) {
        memset(cnt, 0, sizeof(cnt));
        for (int i = 0; i < n; i++) {
            cnt[(ua[i] >> shift) & 0xFF]++;
        }

        if (shift == 24) {
            int pos[256];
            pos[128] = 0;
            for (int i = 129; i < 256; i++) {
                pos[i] = pos[i-1] + cnt[i-1];
            }
            pos[0] = pos[255] + cnt[255];
            for (int i = 1; i < 128; i++) {
                pos[i] = pos[i-1] + cnt[i-1];
            }

            for (int i = 0; i < n; i++) {
                int b = (ua[i] >> 24) & 0xFF;
                ut[pos[b]++] = ua[i];
            }
        } else {
            int pos[256];
            pos[0] = 0;
            for (int i = 1; i < 256; i++) pos[i] = pos[i-1] + cnt[i-1];

            for (int i = 0; i < n; i++) {
                int b = (ua[i] >> shift) & 0xFF;
                ut[pos[b]++] = ua[i];
            }
        }
        unsigned int* temp = ua;
        ua = ut;
        ut = temp;
    }
    
    if ((int*)ua != a) {
        memcpy(a, ua, n * sizeof(int));
    }
}
###
Thay vì bắt đầu từ 0 đến 255 như các byte thấp, tại byte cao nhất, các giá trị từ 128 đến 255 (đại diện cho số âm trong hệ bù hai) được xếp trước, sau đó mới đến các giá trị từ 0 đến 127 (số dương).

Trong khi đó, bài toán sắp xếp chuỗi theo thứ tự từ điển được giải quyết bằng phương pháp MSD đệ quy. Để tối ưu hóa hiệu suất, hệ thống sử dụng mảng chỉ số phụ nhằm hạn chế tối đa việc sao chép các chuỗi ký tự có độ dài lên tới 100 ký tự, điều vốn gây lãng phí bộ nhớ và làm chậm quá trình hoán vị.

###
void msd_radix_sort(int left, int right, int pos, int* idx, int* tmp) {
    if (left >= right || pos >= 100) return;

    int count[28];
    for (int i = 0; i < 28; i++) count[i] = 0;

    for (int i = left; i <= right; i++) {
        char c = strs_data[idx[i]][pos];
        if (c == '\0') count[0]++;
        else count[c - 'a' + 1]++;
    }

    int start_pos[28];
    start_pos[0] = left;
    for (int i = 1; i < 28; i++) {
        start_pos[i] = start_pos[i - 1] + count[i - 1];
    }

    int current_pos[28];
    for (int i = 0; i < 28; i++) current_pos[i] = start_pos[i];

    for (int i = left; i <= right; i++) {
        char c = strs_data[idx[i]][pos];
        int bucket = (c == '\0') ? 0 : (c - 'a' + 1);
        tmp[current_pos[bucket]++] = idx[i];
    }

    for (int i = left; i <= right; i++) {
        idx[i] = tmp[i];
    }

    for (int i = 1; i < 27; i++) {
        if (count[i] > 1) {
            msd_radix_sort(start_pos[i], start_pos[i] + count[i] - 1, pos + 1, idx, tmp);
        }
    }
}
###

Riêng đối với yêu cầu sắp xếp ưu tiên độ dài, sự kết hợp giữa Bucket Sort để phân loại nhóm độ dài và Radix Sort để sắp xếp thứ tự từ điển bên trong mỗi nhóm đã tạo ra một hệ thống xử lý phân tầng cực kỳ hiệu quả.

###
for(int len = 10; len <= MAXLEN; len++){
        vector<int>& bucket = byLen[len];
        if(bucket.empty()) continue;
        int m = bucket.size();
        
        vector<int> temp(m);
        int cnt[27];
        
        for(int pos = len - 1; pos >= 0; pos--){
            memset(cnt, 0, sizeof(cnt));
            for(int i = 0; i < m; i++) cnt[(unsigned char)strs[bucket[i]][pos] - 'a' + 1]++;
            for(int c = 1; c <= 26; c++) cnt[c] += cnt[c-1];
            for(int i = 0; i < m; i++) temp[cnt[(unsigned char)strs[bucket[i]][pos] - 'a']++] = bucket[i];
            for(int i = 0; i < m; i++) bucket[i] = temp[i];
        }
        
        for(int i = 0; i < m; i++) result.push_back(strs[bucket[i]]);
    }
    
    cout << n << '\n';
    for(auto& s : result) cout << s << '\n';
###

Phương pháp này tận dụng tính chất của bài toán: khi các chuỗi có cùng độ dài, việc áp dụng Radix Sort LSD không cần kiểm tra ký tự kết thúc \0, giúp thuật toán đạt tốc độ tối đa và luôn đảm bảo tính ổn định trong kết quả sắp xếp.

# Cách thức sinh test case trong test_gen.cpp

Để đảm bảo tính đối kháng và thử thách tối đa các giải thuật sắp xếp, tệp sinh test được xây dựng nhằm tạo ra các bộ dữ liệu mang tính phá vỡ các cấu trúc tối ưu thông thường. Do quy định khắt khe về việc chỉ được sử dụng một số thư viện tiêu chuẩn như iostream hay cstring, em đã tự cài đặt bộ sinh số ngẫu nhiên giả lập dựa trên thuật toán LCG để đảm bảo tính ổn định và khả năng tái lập của các bộ test mà không phụ thuộc vào các hàm thư viện bị hạn chế. Bộ test này được thiết kế để tấn công trực diện vào các giải thuật so sánh truyền thống như QuickSort hay MergeSort, đặc biệt là các phiên bản chọn điểm chốt không tối ưu. Đối với các bài toán về chuỗi, chiến thuật trọng tâm là tạo ra các tiền tố chung cực dài để buộc các trình so sánh mặc định phải duyệt qua toàn bộ chiều dài chuỗi mới có thể phân định thứ tự, từ đó làm bùng nổ thời gian thực thi so với các chuỗi ngẫu nhiên. Ngoài ra, các trường hợp mảng đã sắp xếp sẵn hoặc chứa mật độ phần tử trùng lặp cao cũng được lồng ghép để vô hiệu hóa khả năng tối ưu hóa của bài giải, buộc giải thuật phải rơi vào các trường hợp xấu nhất với độ phức tạp O(n^2) 

###
unsigned int m_rd() {
    sd = (1103515245 * sd + 12345) % 2147483648;
    return (unsigned int)sd;
}
###
Đây là thuật toán sinh số ngẫu nhiên giả lập giúp tạo ra dữ liệu mà không cần sử dụng thư viện cstdlib hay ctime vốn không nằm trong danh sách cho phép. Việc sử dụng một hằng số cố định làm hạt giống giúp các bộ test luôn đồng nhất trong mọi lần thực thi.

###
void t_sl(int stt) {
    int n = 100000;
    cout << n << "\n";
    if (stt == 1) {
        for (int i = 0; i < n; i++) cout << s_ch(100) << "\n";
    } 
    // Bắt đầu từ đây --------------------------------
    else if (stt == 2) {
        string t_to = s_ch(90);
        for (int i = 0; i < n; i++) cout << t_to << s_ch(10) << "\n";
    } 
    // -------------------------------- Đến đây
    else if (stt == 3) {
        string s = s_ch(100);
        for (int i = 0; i < n; i++) cout << s << "\n";
    } else if (stt == 4) {
        for (int i = 0; i < n; i++) {
            string s = "";
            for(int j = 0; j < 100; j++) s += (m_rd() % 2 == 0 ? 'a' : 'b');
            cout << s << "\n";
        }
    } else {
        for (int i = 0; i < n; i++) cout << "aaaaaaaaaa" << s_ch(5) << "\n";
    }
}
###
Đoạn mã này thực hiện chiến thuật tạo tiền tố chung dài cho bài toán sắp xếp thứ tự từ điển.
Ở phần được chú thích, bằng cách tạo ra 90 ký tự đầu tiên giống hệt nhau cho toàn bộ 100.000 chuỗi, thuật toán buộc mọi phép so sánh đều phải thực hiện ở những ký tự cuối cùng, gây áp lực cực lớn lên các giải thuật dựa trên so sánh ký tự đơn thuần.

###
void t_i(int stt) {
    int n = 100000;
    cout << n << "\n";
    if (stt == 1) {
        for (int i = 0; i < n; i++) cout << (int)m_rd() << "\n";
    } 
    // Bắt đầu từ đây --------------------------------
    else if (stt == 2) {
        vector<int> v(n);
        for (int i = 0; i < n; i++) v[i] = m_rd();
        m_st(v, 0, n - 1, false);
        for (int i = 0; i < n; i++) cout << v[i] << "\n";
    } 
    // -------------------------------- Đến đây
    else if (stt == 3) {
        vector<int> v(n);
        for (int i = 0; i < n; i++) v[i] = m_rd();
        m_st(v, 0, n - 1, true);
        for (int i = 0; i < n; i++) cout << v[i] << "\n";
    } else if (stt == 4) {
        int x = m_rd();
        for (int i = 0; i < n; i++) cout << x << "\n";
    } else {
        for (int i = 0; i < n; i++) cout << (i % 2 == 0 ? 2147483647 : -2147483648) << "\n";
    }
}
###
Đây là cách thức sinh ra mảng đã được sắp xếp sẵn để kiểm tra tính hiệu quả của các giải thuật phân vùng. Nếu bài giải sử dụng QuickSort với cách chọn điểm chốt ở hai đầu mảng, bộ test này sẽ khiến giải thuật bị suy biến và mất rất nhiều thời gian để hoàn thành lượt sắp xếp.
