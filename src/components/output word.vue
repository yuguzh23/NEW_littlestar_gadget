<script setup>
import { ref } from 'vue';
import * as docx from 'docx';
import littlestarImage from '../assets/images/littlestar.jpg';

// 將圖片轉換為base64格式
const getBase64Image = (imgPath) => {
    return new Promise((resolve, reject) => {
        const img = new Image();
        img.onload = () => {
            const canvas = document.createElement('canvas');
            canvas.width = img.width;
            canvas.height = img.height;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0);
            const dataURL = canvas.toDataURL('image/jpeg');
            resolve(dataURL.split(',')[1]);
        };
        img.onerror = () => reject(new Error('圖片加載失敗'));
        img.src = imgPath;
    });
};
const emit = defineEmits(['success']);

// 匯出簽到表功能
const exportAttendanceSheet = async (selectedDate, yunVolunteers, linVolunteers, selectedVenue, isPreTraining = false) => {
    try {
        // 根據場地選擇志工名單
        let volunteers = [];
        if (selectedVenue === '雲科場' && yunVolunteers) {
            // 確保即使沒有勾選顯示選項，也能正確匯出志工資訊
            volunteers = yunVolunteers.split('\n').filter(v => v.trim());
        }
        if (selectedVenue === '林內場' && linVolunteers) {
            volunteers = volunteers.concat(linVolunteers.split('\n').filter(v => v.trim()));
        }

        // 計算日期和設置活動信息
        let activityDate = selectedDate;
        const dateMatch = selectedDate.match(/(\d+)月(\d+)日/);
        if (dateMatch) {
            const month = parseInt(dateMatch[1]); // 使用實際月份
            const day = parseInt(dateMatch[2]);
            // 根據前訓選項設置星期
            const weekday = isPreTraining ? '三' : '六';
            activityDate = `${month}月${day}日(${weekday})`;
        }
        let activityName = '小星星週六課輔';
        let activityTime = '12:00-13:00';
        let activityLocation = '';

        if (isPreTraining) {
            // 計算前三天的日期
            const date = new Date();
            // 解析月日格式，例如「5月20日」
            const dateMatch = selectedDate.match(/(\d+)月(\d+)日/);
            if (dateMatch) {
                const month = parseInt(dateMatch[1]) - 1; // 月份從0開始
                const day = parseInt(dateMatch[2]);
                date.setMonth(month);
                date.setDate(day);
            }
            date.setDate(date.getDate() - 3);
            // 格式化為「月日(星期三)」格式
            activityDate = `${date.getMonth() + 1}月${date.getDate()}日(三)`;

            // 根據場地設置前訓信息
            if (selectedVenue === '雲科場') {
                activityName += '-雲科場前訓';
                activityTime = '12:00-13:00';
                activityLocation = '活動中心(GA244)';
            } else if (selectedVenue === '林內場') {
                activityName += '-林內場前訓';
                activityTime = '12:00-13:00';
                activityLocation = '林內國小(GA248)';
            }
        } else {
            // 一般課輔信息
            if (selectedVenue === '雲科場') {
                activityName += '-雲科場';
                activityTime = '8:30-17:00';
                activityLocation = '活動中心(GA132)';
            } else if (selectedVenue === '林內場') {
                activityName += '-林內場';
                activityTime = '8:25';
                activityLocation = '林內國小';
            }
        }

        // 使用固定的頁首內容，根據用戶需求設置
        const doc = new docx.Document({
            sections: [{
                properties: {
                    tableLayout: {
                        type: docx.TableLayoutType.FIXED,
                    },
                    page: {
                        size: {
                            width: docx.convertMillimetersToTwip(210),
                            height: docx.convertMillimetersToTwip(297),
                        },
                        margin: {
                            top: docx.convertMillimetersToTwip(25),
                            right: docx.convertMillimetersToTwip(20),
                            bottom: docx.convertMillimetersToTwip(20),
                            left: docx.convertMillimetersToTwip(20),
                        },
                    },
                },
                // 設置頁首
                headers: {
                    default: new docx.Header({
                        children: [
                            new docx.Paragraph({
                                children: [
                                    new docx.TextRun({
                                        text: '小星星週六課輔志工簽到表',
                                        bold: true,
                                        size: 40,
                                        font: '微軟正黑體'
                                    })
                                ],
                                alignment: docx.AlignmentType.CENTER,
                                spacing: { after: 400 }
                            }),
                            new docx.Paragraph({
                                children: [
                                    new docx.TextRun({
                                        text: `● 活動: ${activityName}`,
                                        size: 28,
                                        bold: true,
                                        font: '微軟正黑體'
                                    })
                                ],
                                alignment: docx.AlignmentType.LEFT
                            }),
                            new docx.Paragraph({
                                children: [
                                    new docx.TextRun({
                                        text: `● 時間: ${activityDate} ${activityTime}`,
                                        size: 28,
                                        bold: true,
                                        font: '微軟正黑體'
                                    })
                                ],
                                alignment: docx.AlignmentType.LEFT
                            }),
                            new docx.Paragraph({
                                children: [
                                    new docx.TextRun({
                                        text: `● 地點: ${activityLocation}`,
                                        size: 28,
                                        bold: true,
                                        font: '微軟正黑體'
                                    })
                                ],
                                alignment: docx.AlignmentType.LEFT
                            })
                        ]
                    })
                },
                footers: {
                    default: new docx.Footer({
                        children: [
                            new docx.Paragraph({
                                children: [
                                    new docx.ImageRun({
                                        data: await getBase64Image(littlestarImage),
                                        transformation: {
                                            width: 300,
                                            height: 65
                                        },
                                        floating: {
                                            horizontalPosition: {
                                                offset: 0,
                                                align: docx.HorizontalPositionAlign.CENTER
                                            },
                                            verticalPosition: {
                                                offset: -7000000,
                                                align: docx.VerticalPositionAlign.BOTTOM
                                            }
                                        }
                                    })
                                ],
                                alignment: docx.AlignmentType.CENTER,
                                spacing: { before: 200 }
                            })
                        ]
                    })
                },
                children: [
                    ...createSignatureTable(volunteers.length > 0 ? volunteers : [])
                ]
            }]
        });

        // 生成文檔
        const blob = await docx.Packer.toBlob(doc);
        const url = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = url;
        link.download = `小星星志工簽到表_${selectedDate}.docx`;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(url);

        // 發送成功事件
        emit('success', '簽到表已成功匯出！');
    } catch (error) {
        console.error('匯出簽到表時發生錯誤：', error);
    }
};

// 創建簽名表格
const createSignatureTable = (volunteers) => {
    // 如果志工人數超過15人，需要分頁處理
    const tables = [];
    const maxVolunteersPerPage = 15;
    const totalPages = Math.ceil(volunteers.length / maxVolunteersPerPage);

    for (let pageIndex = 0; pageIndex < totalPages; pageIndex++) {
        const startIndex = pageIndex * maxVolunteersPerPage;
        const endIndex = Math.min(startIndex + maxVolunteersPerPage, volunteers.length);
        const pageVolunteers = volunteers.slice(startIndex, endIndex);

        const rows = [
        // 表頭
        new docx.TableRow({
            height: { value: 600, rule: docx.HeightRule.ATLEAST },
            children: [
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '序號', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 1000, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '姓名', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 2000, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '綽號', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 2000, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '簽到(名字)', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 2500, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '簽到時間', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 2000, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                }),
                new docx.TableCell({
                    children: [new docx.Paragraph({
                        spacing: { before: 120, after: 120 },
                        children: [new docx.TextRun({ text: '簽退時間', size: 28, bold: true, font: '微軟正黑體' })],
                        alignment: docx.AlignmentType.CENTER})],
                    width: { size: 2000, type: docx.WidthType.DXA },
                    verticalAlign: docx.VerticalAlign.CENTER
                })
            ]
        })
    ];

    // 添加志工行
    pageVolunteers.forEach((volunteer, index) => {
        // 解析志工資訊，支援「姓名(綽號)-代號」格式，並移除代號
        let name = volunteer;
        let nickname = '';
        
        // 先處理原始字串，移除方括號中的代號 [xxx] 和 -代號 格式
        let processedVolunteer = volunteer.replace(/\[.*?\]/g, '').trim(); // 移除 [xxx]
        processedVolunteer = processedVolunteer.replace(/\s*-.*$/, '').trim(); // 移除 -代號
        
        // 檢查是否有括號內的綽號
        const nameWithNickname = processedVolunteer.match(/^(.+?)\s*\((.+?)\)\s*$/);
        if (nameWithNickname) {
            // 如果匹配到「姓名(綽號)」格式
            name = nameWithNickname[1].trim();
            nickname = nameWithNickname[2].trim();
        } else {
            // 如果沒有括號，則整個字串視為姓名
            name = processedVolunteer.trim();
            nickname = '';
        }
        rows.push(
            new docx.TableRow({
                height: { value: 600, rule: docx.HeightRule.ATLEAST },
                children: [
                    new docx.TableCell({
                        children: [new docx.Paragraph({
                            spacing: { before: 120, after: 120 },
                            children: [new docx.TextRun({ text: (index + 1).toString(), size: 28, bold: true, font: '微軟正黑體' })],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 1000, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                    new docx.TableCell({
                        children: [new docx.Paragraph({
                            spacing: { before: 120, after: 120 },
                            children: [new docx.TextRun({ text: name, size: 28, bold: true, font: '微軟正黑體' })],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 2000, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                    new docx.TableCell({
                        children: [new docx.Paragraph({
                            spacing: { before: 120, after: 120 },
                            children: [new docx.TextRun({ text: nickname, size: 28, bold: true, font: '微軟正黑體' })],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 2000, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                    new docx.TableCell({
                        children: [new docx.Paragraph({ 
                            spacing: { before: 120, after: 120 },
                            children: [],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 2500, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                    new docx.TableCell({
                        children: [new docx.Paragraph({
                            spacing: { before: 120, after: 120 }, 
                            children: [],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 2000, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                    new docx.TableCell({
                        children: [new docx.Paragraph({
                            spacing: { before: 120, after: 120 }, 
                            children: [],
                            alignment: docx.AlignmentType.CENTER})],
                        width: { size: 2000, type: docx.WidthType.DXA },
                        verticalAlign: docx.VerticalAlign.CENTER
                    }),
                ]
            })
        );
    });

        const table = new docx.Table({
        rows: rows,
        width: { size: 8500, type: docx.WidthType.DXA },
        borders: {
            top: { style: docx.BorderStyle.SINGLE, size: 4 },
            bottom: { style: docx.BorderStyle.SINGLE, size: 4 },
            left: { style: docx.BorderStyle.SINGLE, size: 4 },
            right: { style: docx.BorderStyle.SINGLE, size: 4 },
            insideHorizontal: { style: docx.BorderStyle.SINGLE, size: 2 },
            insideVertical: { style: docx.BorderStyle.SINGLE, size: 2 }
        },
        tableProperties: {
            tableLayout: docx.TableLayoutType.FIXED,
            alignment: docx.AlignmentType.CENTER,
            preferredWidth: 8500,
            preferredWidthType: docx.WidthType.DXA,
            cantSplit: true,
            tableIndent: {
                size: 0,
                type: docx.WidthType.DXA
            }
        },
        columnWidths: [1000, 1400, 1400, 2200, 1500, 1500],
        // 設置表格行高
        layout: docx.TableLayoutType.FIXED
        });
        tables.push(table);

        // 如果不是最後一頁，添加分頁符
        if (pageIndex < totalPages - 1) {
            tables.push(
                new docx.Paragraph({
                    children: [new docx.PageBreak()]
                })
            );
        }
    }

    return tables;
};

// 導出函數
defineExpose({
    exportAttendanceSheet
});
</script>

<template>
</template>