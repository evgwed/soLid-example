```php
<?php

class ScheduleDto
{
    /** @var string */
    private $stationCode;
    /** @var string */
    private $startTime;
    /** @var string */
    private $endTime;

    public function __construct(string $stationCode, string $startTime, string $endTime)
    {
        $this->stationCode = $stationCode;
        $this->startTime = $startTime;
        $this->endTime = $endTime;
    }

    public function getStationCode(): string
    {
        return $this->stationCode;
    }

    public function getStartTime(): string
    {
        return $this->startTime;
    }

    public function getEndTime(): string
    {
        return $this->endTime;
    }

    public function toArray(): array
    {
        return [
            'stationCode' => $this->stationCode,
            'startTime' => $this->startTime,
            'endTime' => $this->endTime,
        ];
    }
}

class SuperScheduleDto extends ScheduleDto
{
    /** @var string */
    private $director;

    public function __construct(string $stationCode, string $startTime, string $endTime, string $director)
    {
        parent::__construct($stationCode, $startTime, $endTime);
        $this->director = $director;
    }

    public function getDirector(): string
    {
        return $this->director;
    }

    public function toArray(): array
    {
         return parent::toArray() + ['director' => $this->director];
    }
}

interface DataProviderContract
{
    /**
     * @return ScheduleDto[]
     */
    public function getScheduleItems(): array;
}


class GIS2DataProvider implements DataProviderContract
{
    public function getScheduleItems(): array
    {
        $response = $this->getDataFromServer();

        return $this->serialize($response);
    }

    protected function serialize(array $dataFromServer): array
    {
        return array_map(function (array $dataItem) {
            return new ScheduleDto(
                $dataItem['station'],
                $dataItem['start'],
                $dataItem['end']
            );
        }, $dataFromServer);
    }

    protected function getDataFromServer(): array
    {
        return [
            [
                'station' => 'BWM Motors',
                'start' => '08:00',
                'end' => '17:00'
            ],
            [
                'station' => 'Avto73',
                'start' => '09:00',
                'end' => '18:00'
            ]
        ];
    }
}

class ScheduleRepository
{
    public function save(ScheduleDto $item): void
    {
        // save logic
        echo $this->saveLog($item) . PHP_EOL;
    }

    protected function saveLog(ScheduleDto $item): string
    {
        return 'saved ' . $item->getStationCode();
    }
}

class ScheduleParserService
{
    /** @var DataProviderContract */
    private $dataProvider;
    /** @var ScheduleRepository */
    private $scheduleRepository;

    public function __construct(DataProviderContract $dataProvider, ScheduleRepository $scheduleRepository)
    {
        $this->dataProvider = $dataProvider;
        $this->scheduleRepository = $scheduleRepository;
    }

    public function parse(): void
    {
        $items = $this->dataProvider->getScheduleItems();
        foreach ($items as $item) {
            $this->scheduleRepository->save($item);
        }
    }
}

class GoogleDataProvider implements DataProviderContract
{
    public function getScheduleItems(): array
    {
        return [
            new ScheduleDto(
                'Googled BWM Motors',
                '08:00',
                '17:00'
            ),

            new ScheduleDto(
                'Googled Avto73',
                '09:00',
                '18:00'
            )
        ];
    }
}

class AdvancedScheduleRepository extends ScheduleRepository
{
    protected function saveLog(ScheduleDto $item): string
    {
        return json_encode($item->toArray());
    }
}

class Advanced2GISDataProvider extends GIS2DataProvider
{
    protected function serialize(array $dataFromServer): array
    {
        return array_map(function (array $dataItem) {
            return new ScheduleDto(
                $dataItem['station'],
                $this->prepareTime($dataItem['start']),
                $this->prepareTime($dataItem['end'])
            );
        }, $dataFromServer);
    }

    protected function prepareTime(string $time): string
    {
        return str_replace(':', '.', $time);
    }
}

class SuperAdvanced2GISDataProvider extends Advanced2GISDataProvider
{
    protected function serialize(array $dataFromServer): array
    {
        return array_map(function (array $dataItem) {
            return new SuperScheduleDto(
                $dataItem['station'],
                $this->prepareTime($dataItem['start']),
                $this->prepareTime($dataItem['end']),
                $dataItem['director'] ?? 'none'
            );
        }, $dataFromServer);
    }

    protected function getDataFromServer(): array
    {
        return [
            [
                'station' => 'BWM Motors',
                'start' => '08:00',
                'end' => '17:00',
                'director' => 'Klava Koka'
            ],
            [
                'station' => 'Avto73',
                'start' => '09:00',
                'end' => '18:00'
            ]
        ];
    }
}

$di = [
//    'scheduleRepository' => ScheduleRepository::class,
    'scheduleRepository' => AdvancedScheduleRepository::class,
//    'dataProvider' => GoogleDataProvider::class,
//    'dataProvider' => GIS2DataProvider::class,
//    'dataProvider' => Advanced2GISDataProvider::class,
    'dataProvider' => SuperAdvanced2GISDataProvider::class,
];

$parser = new ScheduleParserService(new $di['dataProvider'], new $di['scheduleRepository']);
$parser->parse();



```
